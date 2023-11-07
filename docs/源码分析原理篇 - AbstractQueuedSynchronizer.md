# 内部属性

```java
/**
 * The synchronization state.
 * 控制加锁、解锁
 */
private volatile int state;
/**
 * Head of the wait queue, lazily initialized.  Except for
 * initialization, it is modified only via method setHead.  Note:
 * If head exists, its waitStatus is guaranteed not to be
 * CANCELLED.
 * 
 * 等待队列的头节点
 */
private transient volatile Node head;

/**
 * Tail of the wait queue, lazily initialized.  Modified only via
 * method enq to add new wait node.
 * 
 * 等待队列的尾节点
 */
private transient volatile Node tail;
```

# Node 类

```java
/**
 * 队列节点
 */
static final class Node {
	/** Marker to indicate a node is waiting in shared mode */
	static final Node SHARED = new Node();
	/** Marker to indicate a node is waiting in exclusive mode */
	static final Node EXCLUSIVE = null;

	/** waitStatus value to indicate thread has cancelled */
	static final int CANCELLED =  1;
	/** waitStatus value to indicate successor's thread needs unparking */
	static final int SIGNAL    = -1;
	/** waitStatus value to indicate thread is waiting on condition */
	static final int CONDITION = -2;
	/**
	 * waitStatus value to indicate the next acquireShared should
	 * unconditionally propagate
	 */
	static final int PROPAGATE = -3;

	/**
	 * Status field, taking on only the values:
	 *   SIGNAL:     The successor of this node is (or will soon be)
	 *               blocked (via park), so the current node must
	 *               unpark its successor when it releases or
	 *               cancels. To avoid races, acquire methods must
	 *               first indicate they need a signal,
	 *               then retry the atomic acquire, and then,
	 *               on failure, block.
	 *   CANCELLED:  This node is cancelled due to timeout or interrupt.
	 *               Nodes never leave this state. In particular,
	 *               a thread with cancelled node never again blocks.
	 *   CONDITION:  This node is currently on a condition queue.
	 *               It will not be used as a sync queue node
	 *               until transferred, at which time the status
	 *               will be set to 0. (Use of this value here has
	 *               nothing to do with the other uses of the
	 *               field, but simplifies mechanics.)
	 *   PROPAGATE:  A releaseShared should be propagated to other
	 *               nodes. This is set (for head node only) in
	 *               doReleaseShared to ensure propagation
	 *               continues, even if other operations have
	 *               since intervened.
	 *   0:          None of the above
	 *
	 * The values are arranged numerically to simplify use.
	 * Non-negative values mean that a node doesn't need to
	 * signal. So, most code doesn't need to check for particular
	 * values, just for sign.
	 *
	 * The field is initialized to 0 for normal sync nodes, and
	 * CONDITION for condition nodes.  It is modified using CAS
	 * (or when possible, unconditional volatile writes).
	 */
	volatile int waitStatus;

	/**
	 * 前置节点
	 */
	volatile Node prev;

	/**
	 * 后置节点
	 */
	volatile Node next;

	/**
	 * 节点代表的线程
	 */
	volatile Thread thread;

	/**
	 * Link to next node waiting on condition, or the special
	 * value SHARED.  Because condition queues are accessed only
	 * when holding in exclusive mode, we just need a simple
	 * linked queue to hold nodes while they are waiting on
	 * conditions. They are then transferred to the queue to
	 * re-acquire. And because conditions can only be exclusive,
	 * we save a field by using special value to indicate shared
	 * mode.
	 */
	Node nextWaiter;

	Node() {    // Used to establish initial head or SHARED marker
	}

	Node(Thread thread, Node mode) {     // Used by addWaiter
		this.nextWaiter = mode;
		this.thread = thread;
	}

	Node(Thread thread, int waitStatus) { // Used by Condition
		this.waitStatus = waitStatus;
		this.thread = thread;
	}
}
```

# ConditionObject 类

```java
/**
 * 条件变量
 */
public class ConditionObject implements Condition, java.io.Serializable {
	private static final long serialVersionUID = 1173984872572414699L;
	/** First node of condition queue. */
	private transient Node firstWaiter;
	/** Last node of condition queue. */
	private transient Node lastWaiter;

	/**
	 * Creates a new {@code ConditionObject} instance.
	 */
	public ConditionObject() { }
}
```

# acquireQueued(**final** Node node, **int** arg)

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            // 前置节点是头节点则尝试获取锁（调用子类tryAcquire）
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 获取锁失败或前置节点不是头节点，则将前置节点的waitStatus设为Node.SIGNAL，然后阻塞线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                // parkAndCheckInterrupt()返回true，表示线程是被中断唤醒
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	int ws = pred.waitStatus;
	if (ws == Node.SIGNAL)
		/*
		 * This node has already set status asking a release
		 * to signal it, so it can safely park.
		 */
		return true;
	if (ws > 0) {
		/*
		 * Predecessor was cancelled. Skip over predecessors and
		 * indicate retry.
		 */
        // 删除已取消的前置节点
		do {
			node.prev = pred = pred.prev;
		} while (pred.waitStatus > 0);
		pred.next = node;
	} else {
		/*
		 * waitStatus must be 0 or PROPAGATE.  Indicate that we
		 * need a signal, but don't park yet.  Caller will need to
		 * retry to make sure it cannot acquire before parking.
		 */
        // 将前置节点的waitStatus设为Node.SIGNAL，让它释放锁的时候记得唤醒一下自己
		compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	}
	return false;
}
```

# release(int arg)

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

/**
 * Wakes up node's successor, if one exists.
 *
 * @param node the node
 */
private void unparkSuccessor(Node node) {
	/*
	 * If status is negative (i.e., possibly needing signal) try
	 * to clear in anticipation of signalling.  It is OK if this
	 * fails or if status is changed by waiting thread.
	 */
	int ws = node.waitStatus;
	if (ws < 0)
		compareAndSetWaitStatus(node, ws, 0);

	/*
	 * Thread to unpark is held in successor, which is normally
	 * just the next node.  But if cancelled or apparently null,
	 * traverse backwards from tail to find the actual
	 * non-cancelled successor.
	 */
	Node s = node.next;
	if (s == null || s.waitStatus > 0) {
		s = null;
        // 从队列尾开始，从后往前找waitStatus<=0的节点来唤醒
        // 为什么从队列尾开始找？
        // 为什么找waitStatus<=0的节点？
        // 不会出现唤醒后加锁失败的问题？毕竟前置节点可能不是头结点
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)
				s = t;
	}
	if (s != null)
		LockSupport.unpark(s.thread);
}
```

