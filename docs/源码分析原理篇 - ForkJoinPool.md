# ForkJoinPool

## 内部属性

```java
// Instance fields

// AC+TC+SS+ID
// AC(活跃worker数)
// TC(总Worker数)
// SS(WorkQueue状态)
// ID(WorkQueue在workQueues数组的下标)
volatile long ctl;                   // main pool control
// 线程池状态+锁状态
volatile int runState;               // lockable status
// 并行度+异步模式(FILO或FIFO，默认是FILO，即栈的方式)
final int config;                    // parallelism, mode
int indexSeed;                       // to generate worker index
// 工作队列数组
volatile WorkQueue[] workQueues;     // main registry
// 线程工厂
final ForkJoinWorkerThreadFactory factory;
// 异常处理器
final UncaughtExceptionHandler ueh;  // per-worker UEH
// 工作线程名称前缀
final String workerNamePrefix;       // to create worker name string
volatile AtomicLong stealCounter;    // also used as sync monitor
```

## 构造方法

```java
/**
 * Creates a {@code ForkJoinPool} with the given parameters.
 *
 * @param parallelism the parallelism level. For default value,
 * use {@link java.lang.Runtime#availableProcessors}.
 * @param factory the factory for creating new threads. For default value,
 * use {@link #defaultForkJoinWorkerThreadFactory}.
 * @param handler the handler for internal worker threads that
 * terminate due to unrecoverable errors encountered while executing
 * tasks. For default value, use {@code null}.
 * @param asyncMode if true,
 * establishes local first-in-first-out scheduling mode for forked
 * tasks that are never joined. This mode may be more appropriate
 * than default locally stack-based mode in applications in which
 * worker threads only process event-style asynchronous tasks.
 * For default value, use {@code false}.
 * @throws IllegalArgumentException if parallelism less than or
 *         equal to zero, or greater than implementation limit
 * @throws NullPointerException if the factory is null
 * @throws SecurityException if a security manager exists and
 *         the caller is not permitted to modify threads
 *         because it does not hold {@link
 *         java.lang.RuntimePermission}{@code ("modifyThread")}
 */
public ForkJoinPool(int parallelism,
					ForkJoinWorkerThreadFactory factory,
					UncaughtExceptionHandler handler,
					boolean asyncMode) {
	this(checkParallelism(parallelism),
		 checkFactory(factory),
		 handler,
		 asyncMode ? FIFO_QUEUE : LIFO_QUEUE,
		 "ForkJoinPool-" + nextPoolId() + "-worker-");
	checkPermission();
}

/**
 * Creates a {@code ForkJoinPool} with the given parameters, without
 * any security checks or parameter validation.  Invoked directly by
 * makeCommonPool.
 */
private ForkJoinPool(int parallelism,
					 ForkJoinWorkerThreadFactory factory,
					 UncaughtExceptionHandler handler,
					 int mode,
					 String workerNamePrefix) {
	this.workerNamePrefix = workerNamePrefix;
	this.factory = factory;
	this.ueh = handler;
	this.config = (parallelism & SMASK) | mode;
	long np = (long)(-parallelism); // offset ctl counts
	this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
}
```

## externalPush()+externalSubmit()

通过 submit、execute 和 invoke 提交任务，最终都会调用 externalPush() 方法，将任务存放到 workQueues 数组中的共享队列

```java
/**
 * Tries to add the given task to a submission queue at
 * submitter's current queue. Only the (vastly) most common path
 * is directly handled in this method, while screening for need
 * for externalSubmit.
 *
 * @param task the task. Caller must ensure non-null.
 */
final void externalPush(ForkJoinTask<?> task) {
	WorkQueue[] ws; WorkQueue q; int m;
    // 每个线程生成的随机数不一样
	int r = ThreadLocalRandom.getProbe();
	int rs = runState;
    // 将任务哈希分配到workQueues数组中的共享队列
	if ((ws = workQueues) != null && (m = (ws.length - 1)) >= 0 &&
        // “& SQMASK” 操作能确保是偶数下标的共享队列
        // “SQMASK”的值是“0x007e”，即126，任何数与之相“与”，都会变成偶数，且不会超过126，能限制偶数下标的槽不超过64个
		(q = ws[m & r & SQMASK]) != null && r != 0 && rs > 0 &&
        // 加锁
		U.compareAndSwapInt(q, QLOCK, 0, 1)) {
		ForkJoinTask<?>[] a; int am, n, s;
		if ((a = q.array) != null &&
			(am = a.length - 1) > (n = (s = q.top) - q.base)) {
            // 将任务存放在 array 数组 的 top 位置
			int j = ((am & s) << ASHIFT) + ABASE;
			U.putOrderedObject(a, j, task);
			U.putOrderedInt(q, QTOP, s + 1);
			U.putIntVolatile(q, QLOCK, 0);
            // 为什么要这个条件才触发worker线程？
			if (n <= 1)
				signalWork(ws, q);
			return;
		}
        // 解锁
		U.compareAndSwapInt(q, QLOCK, 1, 0);
	}
    // 如果workQueues为空或存放的共享队列为空，则调用完整版的方法
	externalSubmit(task);
}

/**
 * Full version of externalPush, handling uncommon cases, as well
 * as performing secondary initialization upon the first
 * submission of the first task to the pool.  It also detects
 * first submission by an external thread and creates a new shared
 * queue if the one at index if empty or contended.
 *
 * @param task the task. Caller must ensure non-null.
 */
private void externalSubmit(ForkJoinTask<?> task) {
	int r;                                    // initialize caller's probe
	if ((r = ThreadLocalRandom.getProbe()) == 0) {
		ThreadLocalRandom.localInit();
		r = ThreadLocalRandom.getProbe();
	}
    // 这是个循环，先初始化workQueues，再初始化共享队列，然后存放任务
	for (;;) {
        // 变量 m 是 “length-1”，变量 k 是随机数
		WorkQueue[] ws; WorkQueue q; int rs, m, k;
        // move 表示重新生成随机数
		boolean move = false;
		if ((rs = runState) < 0) {
			tryTerminate(false, false);     // help terminate
			throw new RejectedExecutionException();
		}
        // 初始化workQueues数组
		else if ((rs & STARTED) == 0 ||     // initialize
				 ((ws = workQueues) == null || (m = ws.length - 1) < 0)) {
			int ns = 0;
            // 加锁（操作 workQueues 数组时加的锁是 runState）
			rs = lockRunState();
			try {
				if ((rs & STARTED) == 0) {
					U.compareAndSwapObject(this, STEALCOUNTER, null,
										   new AtomicLong());
					// create workQueues array with size a power of two
					int p = config & SMASK; // ensure at least 2 slots
					int n = (p > 1) ? p - 1 : 1;
					n |= n >>> 1; n |= n >>> 2;  n |= n >>> 4;
					n |= n >>> 8; n |= n >>> 16; n = (n + 1) << 1;
					workQueues = new WorkQueue[n];
					ns = STARTED;
				}
			} finally {
                // 解锁
				unlockRunState(rs, (rs & ~RSLOCK) | ns);
			}
		}
        // 初始化共享队列的array数组，然后存放任务到 array->top
		else if ((q = ws[k = r & m & SQMASK]) != null) {
            // 加锁（操作 array 数组时加的锁是 QLOCK）
			if (q.qlock == 0 && U.compareAndSwapInt(q, QLOCK, 0, 1)) {
				ForkJoinTask<?>[] a = q.array;
				int s = q.top;
				boolean submitted = false; // initial submission or resizing
				try {                      // locked version of push
					if ((a != null && a.length > s + 1 - q.base) ||
                        // 初始化共享队列的 array 数组
						(a = q.growArray()) != null) {
						int j = (((a.length - 1) & s) << ASHIFT) + ABASE;
						U.putOrderedObject(a, j, task);
						U.putOrderedInt(q, QTOP, s + 1);
						submitted = true;
					}
				} finally {
                    // 解锁
					U.compareAndSwapInt(q, QLOCK, 1, 0);
				}
				if (submitted) {
                    // 添加任务后，就要添加工作线程去执行
					signalWork(ws, q);
					return;
				}
			}
			move = true;                   // move on failure
		}
        // 初始化共享队列
		else if (((rs = runState) & RSLOCK) == 0) { // create new queue
            // 共享队列中的owner线程是空的
			q = new WorkQueue(this, null);
			q.hint = r;
			q.config = k | SHARED_QUEUE;
			q.scanState = INACTIVE;
			rs = lockRunState();           // publish index
			if (rs > 0 &&  (ws = workQueues) != null &&
				k < ws.length && ws[k] == null)
				ws[k] = q;                 // else terminated
			unlockRunState(rs, rs & ~RSLOCK);
		}
		else
			move = true;                   // move if busy
		if (move)
			r = ThreadLocalRandom.advanceProbe(r);
	}
}
```

## pollSubmission()

```java
/**
 * Removes and returns the next unexecuted submission if one is
 * available.  This method may be useful in extensions to this
 * class that re-assign work in systems with multiple pools.
 *
 * @return the next submission, or {@code null} if none
 */
protected ForkJoinTask<?> pollSubmission() {
	WorkQueue[] ws; WorkQueue w; ForkJoinTask<?> t;
	if ((ws = workQueues) != null) {
        // poll workQueues中偶数下标的队列
		for (int i = 0; i < ws.length; i += 2) {
			if ((w = ws[i]) != null && (t = w.poll()) != null)
				return t;
		}
	}
	return null;
}
```

## signalWork(WorkQueue[] ws, WorkQueue q)

创建新的 Worker 线程

## registerWorker(ForkJoinWorkerThread wt)

```java
/**
 * Callback from ForkJoinWorkerThread constructor to establish and
 * record its WorkQueue.
 *
 * @param wt the worker thread
 * @return the worker's queue
 */
final WorkQueue registerWorker(ForkJoinWorkerThread wt) {
	UncaughtExceptionHandler handler;
	wt.setDaemon(true);                           // configure thread
	if ((handler = ueh) != null)
		wt.setUncaughtExceptionHandler(handler);
	WorkQueue w = new WorkQueue(this, wt);
	int i = 0;                                    // assign a pool index
	int mode = config & MODE_MASK;
	int rs = lockRunState();
	try {
		WorkQueue[] ws; int n;                    // skip if no array
		if ((ws = workQueues) != null && (n = ws.length) > 0) {
			int s = indexSeed += SEED_INCREMENT;  // unlikely to collide
			int m = n - 1;
            // “| 1”运算可以确保是奇数下标
			i = ((s << 1) | 1) & m;               // odd-numbered indices
			if (ws[i] != null) {                  // collision
				int probes = 0;                   // step by approx half n
				int step = (n <= 4) ? 2 : ((n >>> 1) & EVENMASK) + 2;
				while (ws[i = (i + step) & m] != null) {
					if (++probes >= n) {
                        // workQueues 中没有空的工作队列，则扩容
						workQueues = ws = Arrays.copyOf(ws, n <<= 1);
						m = n - 1;
						probes = 0;
					}
				}
			}
			w.hint = s;                           // use as random seed
			w.config = i | mode;
			w.scanState = i;                      // publication fence
			ws[i] = w;
		}
	} finally {
		unlockRunState(rs, rs & ~RSLOCK);
	}
	wt.setName(workerNamePrefix.concat(Integer.toString(i >>> 1)));
    // 返回新创建的 WorkQueue，接下来会被赋予给 ForkJoinWorkerThread.workQueue
	return w;
}

protected ForkJoinWorkerThread(ForkJoinPool pool) {
    // Use a placeholder until a useful name can be set in registerWorker
    super("aForkJoinWorkerThread");
    this.pool = pool;
    this.workQueue = pool.registerWorker(this);
}
```

## runWorker(WorkQueue w)

```java
    final void runWorker(WorkQueue w) {
        w.growArray();                   // allocate queue
        int seed = w.hint;               // initially holds randomization hint
        int r = (seed == 0) ? 1 : seed;  // avoid 0 for xorShift
        for (ForkJoinTask<?> t;;) {
            if ((t = scan(w, r)) != null)
                w.runTask(t);
            else if (!awaitWork(w, r))
                break;
            r ^= r << 13; r ^= r >>> 17; r ^= r << 5; // xorshift
        }
    }

```

# WorkQueue 类

## 内部属性

```java
// Instance fields
volatile int scanState;    // versioned, <0: inactive; odd:scanning
int stackPred;             // pool stack (ctl) predecessor
int nsteals;               // number of steals
int hint;                  // randomization and stealer index hint
int config;                // pool index and mode
volatile int qlock;        // 1: locked, < 0: terminate; else 0
volatile int base;         // index of next slot for poll
int top;                   // index of next slot for push
ForkJoinTask<?>[] array;   // the elements (initially unallocated)
final ForkJoinPool pool;   // the containing pool (may be null)
final ForkJoinWorkerThread owner; // owning thread or null if shared
volatile Thread parker;    // == owner during call to park; else null
volatile ForkJoinTask<?> currentJoin;  // task being joined in awaitJoin
volatile ForkJoinTask<?> currentSteal; // mainly used by helpStealer
```

## push()

```java
/**
 * Pushes a task. Call only by owner in unshared queues.  (The
 * shared-queue version is embedded in method externalPush.)
 *
 * @param task the task. Caller must ensure non-null.
 * @throws RejectedExecutionException if array cannot be resized
 */
final void push(ForkJoinTask<?> task) {
	ForkJoinTask<?>[] a; ForkJoinPool p;
	int b = base, s = top, n;
	if ((a = array) != null) {    // ignore if queue removed
		int m = a.length - 1;     // fenced write for task visibility
		U.putOrderedObject(a, ((m & s) << ASHIFT) + ABASE, task);
		U.putOrderedInt(this, QTOP, s + 1);
		if ((n = s - b) <= 1) {
			if ((p = pool) != null)
				p.signalWork(p.workQueues, this);
		}
		else if (n >= m)
            // array 数组扩容
			growArray();
	}
}
```

## pop()

```java
/**
 * Takes next task, if one exists, in LIFO order.  Call only
 * by owner in unshared queues.
 */
final ForkJoinTask<?> pop() {
	ForkJoinTask<?>[] a; ForkJoinTask<?> t; int m;
	if ((a = array) != null && (m = a.length - 1) >= 0) {
		for (int s; (s = top - 1) - base >= 0;) {
			long j = ((m & s) << ASHIFT) + ABASE;
			if ((t = (ForkJoinTask<?>)U.getObject(a, j)) == null)
				break;
			if (U.compareAndSwapObject(a, j, t, null)) {
                // s在上面已经减1了
				U.putOrderedInt(this, QTOP, s);
				return t;
			}
		}
	}
	return null;
}
```

## poll()

```java
/**
 * Takes next task, if one exists, in FIFO order.
 */
final ForkJoinTask<?> poll() {
	ForkJoinTask<?>[] a; int b; ForkJoinTask<?> t;
	while ((b = base) - top < 0 && (a = array) != null) {
		int j = (((a.length - 1) & b) << ASHIFT) + ABASE;
		t = (ForkJoinTask<?>)U.getObjectVolatile(a, j);
		if (base == b) {
			if (t != null) {
				if (U.compareAndSwapObject(a, j, t, null)) {
					base = b + 1;
					return t;
				}
			}
			else if (b + 1 == top) // now empty
				break;
		}
	}
	return null;
}
```

# ForkJoinWorkerThread 类

## 内部属性

```java
/*
 * ForkJoinWorkerThreads are managed by ForkJoinPools and perform
 * ForkJoinTasks. For explanation, see the internal documentation
 * of class ForkJoinPool.
 *
 * This class just maintains links to its pool and WorkQueue.  The
 * pool field is set immediately upon construction, but the
 * workQueue field is not set until a call to registerWorker
 * completes. This leads to a visibility race, that is tolerated
 * by requiring that the workQueue field is only accessed by the
 * owning thread.
 *
 * Support for (non-public) subclass InnocuousForkJoinWorkerThread
 * requires that we break quite a lot of encapsulation (via Unsafe)
 * both here and in the subclass to access and set Thread fields.
 */

final ForkJoinPool pool;                // the pool this thread works in
final ForkJoinPool.WorkQueue workQueue; // work-stealing mechanics
```

## 构造方法

```java
/**
 * Creates a ForkJoinWorkerThread operating in the given pool.
 *
 * @param pool the pool this thread works in
 * @throws NullPointerException if pool is null
 */
protected ForkJoinWorkerThread(ForkJoinPool pool) {
	// Use a placeholder until a useful name can be set in registerWorker
	super("aForkJoinWorkerThread");
	this.pool = pool;
	this.workQueue = pool.registerWorker(this);
}

/**
 * Version for use by the default pool.  This is a separate constructor to
 * avoid affecting the protected constructor.
 */
ForkJoinWorkerThread(ForkJoinPool pool, boolean innocuous) {
	super("aForkJoinWorkerThread");
	if (innocuous) {
		U.putOrderedObject(this, INHERITEDACCESSCONTROLCONTEXT, INNOCUOUS_ACC);
	}
	this.pool = pool;
	this.workQueue = pool.registerWorker(this);
}

/**
 * Version for InnocuousForkJoinWorkerThread
 */
ForkJoinWorkerThread(ForkJoinPool pool, ThreadGroup threadGroup,
					 AccessControlContext acc) {
	super(threadGroup, null, "aForkJoinWorkerThread");
	U.putOrderedObject(this, INHERITEDACCESSCONTROLCONTEXT, acc);
	eraseThreadLocals(); // clear before registering
	this.pool = pool;
	this.workQueue = pool.registerWorker(this);
}
```
