## 如何实现一个线程池

参考优秀博文：https://blog.csdn.net/touch_2011/article/details/6914468

要实现一个线程池，需要具备以下核心要素：

- 任务队列：存放要执行任务
- 工作线程：执行任务
- 线程管理器：创建、销毁、管理工作线程，让工作线程执行任务队列中的任务

以下代码摘取上述博文并做了一定的优化：

```java
/**
 * 线程池类，线程管理器：创建线程，执行任务，销毁线程，获取线程基本信息
 */
final class ThreadPool {

    // 线程池中默认线程的个数为5
    private static int workerNum = 5;
    // 工作线程
    private WorkThread[] workers;
    // 未处理的任务
    private static AtomicInteger finishedTask = new AtomicInteger(0);
    // 任务队列，作为一个缓冲,List线程不安全
    private List<Runnable> workQueue = new LinkedList<>();
    // 单例
    private static ThreadPool threadPool;

    // 创建具有默认线程个数的线程池
    private ThreadPool() {
        this(5);
    }

    // 创建线程池,workerNum为线程池中工作线程的个数
    private ThreadPool(int workerNum) {
        ThreadPool.workerNum = workerNum;
        workers = new WorkThread[workerNum];
        for (int i = 0; i < workerNum; i++) {
            workers[i] = new WorkThread();
            workers[i].start();// 开启线程池中的线程
        }
    }

    // 单态模式，获得一个默认线程个数的线程池
    public static ThreadPool getThreadPool() {
        return getThreadPool(ThreadPool.workerNum);
    }

    /**
     * 单态模式，获得一个指定线程个数的线程池，
     * workerNum > 0 为线程池中工作线程的个数
     * workerNum <= 0 创建默认的工作线程个数
     *
     * @param workerNum
     * @return
     */
    public static ThreadPool getThreadPool(int workerNum) {
        if (workerNum <= 0) {
            workerNum = ThreadPool.workerNum;
        }
        if (threadPool == null) {
            threadPool = new ThreadPool(workerNum);
        }
        return threadPool;
    }

    /**
     * 执行任务,其实只是把任务加入任务队列，什么时候执行由线程池管理器决定
     *
     * @param task
     */
    public void execute(Runnable task) {
        synchronized (workQueue) {
            workQueue.add(task);
            workQueue.notifyAll();
        }
    }

    /**
     * 执行任务,其实只是把任务加入任务队列，什么时候执行由线程池管理器决定
     *
     * @param task
     */
    public void execute(Runnable[] task) {
        synchronized (workQueue) {
            Collections.addAll(workQueue, task);
            workQueue.notifyAll();
        }
    }

    /**
     * 执行任务,其实只是把任务加入任务队列，什么时候执行由线程池管理器决定
     *
     * @param task
     */
    public void execute(List<Runnable> task) {
        synchronized (workQueue) {
            for (Runnable t : task) {
                workQueue.add(t);
            }
            workQueue.notifyAll();
        }
    }

    /**
     * 销毁线程池,该方法保证在所有任务都完成的情况下才销毁所有线程，否则等待任务完成才销毁
     */
    public void shutdown() {
        while (!workQueue.isEmpty()) {// 如果还有任务没执行完成，就先睡会吧
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 工作线程停止工作，且置为null
        for (int i = 0; i < workerNum; i++) {
            workers[i].stopWorker();
            workers[i] = null;
        }
        threadPool = null;
        workQueue.clear();// 清空任务队列
    }

    // 返回工作线程的个数
    public int getWorkThreadNumber() {
        return workerNum;
    }

    // 返回已完成任务的个数,这里的已完成是只出了任务队列的任务个数，可能该任务并没有实际执行完成
    public int getFinishedTasknumber() {
        return finishedTask.get();
    }

    // 返回任务队列的长度，即还没处理的任务个数
    public int getWaitTasknumber() {
        return workQueue.size();
    }

    // 覆盖toString方法，返回线程池信息：工作线程个数和已完成任务个数
    @Override
    public String toString() {
        return "WorkThread number:" + workerNum + "  finished task number:"
               + finishedTask + "  wait task number:" + getWaitTasknumber();
    }

    /**
     * 内部类，工作线程
     */
    private class WorkThread extends Thread {
        // 该工作线程是否有效，用于结束该工作线程
        private boolean isRunning = true;

        /*
         * 关键所在啊，如果任务队列不空，则取出任务执行，若任务队列空，则等待
         */
        @Override
        public void run() {
            Runnable r = null;
            while (isRunning) {// 注意，若线程无效则自然结束run方法，该线程就没用了
                synchronized (workQueue) {
                    while (isRunning && workQueue.isEmpty()) {// 队列为空
                        try {
                            workQueue.wait(20);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    if (!workQueue.isEmpty()) {
                        r = workQueue.remove(0);// 取出任务
                    }
                }
                if (r != null) {
                    r.run();// 执行任务
                }
                finishedTask.getAndIncrement();
                r = null;
            }
        }

        // 停止工作，让该线程自然执行完run方法，自然结束
        public void stopWorker() {
            isRunning = false;
        }
    }
}
```

测试类：

```java
public class Test {

    public static void main(String[] args) {
        // 创建3个线程的线程池
        ThreadPool t = ThreadPool.getThreadPool(3);
        t.execute(new Runnable[] {new Task(), new Task(), new Task()});
        t.execute(new Runnable[] {new Task(), new Task(), new Task()});
        System.out.println(t);
        t.shutdown();// 所有线程都执行完成才destory
        System.out.println(t);
    }

    // 任务类
    static class Task implements Runnable {
        private static AtomicInteger i = new AtomicInteger(1);

        @Override
        public void run() {// 执行任务
            System.out.println("任务 " + i.getAndIncrement() + " 完成");
        }
    }

}
```



## ThreadPoolExecutor 源码分析

