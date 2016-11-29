title: 限制Java线程池运行线程以及等待线程数量的策略
date: 2016/7/5
categories:
- coding
tags:
- java
---
对于`java.util.concurrent.Executors`所提供的`FixedThreadPool`，可以保证可以在内存中有固定数量的线程数运行。但是由于`FixedThreadPool`绑定的是`LinkedBlockingQueue`。队列的上限没有限制（默认上限为`Integer.MAX_VALUE`），不断的提交新的线程，会造成任务在内存中长时间的堆积。

我们有可能面临如下的场景，主线程不断地提交任务线程，希望有固定数量的在线程中运行，也不想造成线程在内存中大量的等待堆积。由此需要我们自己定义一个线程池策略。`ThreadPoolExecutor`为我们线程池的设置提供了很大的灵活性。

首先看`FixedThreadPool`的实现：

```
	public static ExecutorService More ...newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
	        return new ThreadPoolExecutor(nThreads, nThreads,
	                                      0L, TimeUnit.MILLISECONDS,
	                                      new LinkedBlockingQueue<Runnable>(),
	                                      threadFactory);
	    }
```

可以看到，FixedThreadPool绑定的是`LinkedBlockingQueue<Runnable>`。我们需要做的第一个改造就是绑定有大小上线的BlockingQueue，在我的实现中绑定`ArrayBlockingQueue<Runnable>`并设置了size。

第二个是采用CallerRunsPolicy。ThreadPoolExecutor可以定义不同的任务拒绝策略。CallerRunsPolicy指的是当线程池拒绝该任务的时候，线程在本地线程直接`execute`。这样就限制了本地线程的循环提交流程。

```
	BlockingQueue<Runnable> workingQueue = new ArrayBlockingQueue<Runnable>(10);
    RejectedExecutionHandler rejectedExecutionHandler =
        new ThreadPoolExecutor.CallerRunsPolicy();
    ExecutorService threadPool = new ThreadPoolExecutor(10, 10, 0L, TimeUnit.MILLISECONDS,
        workingQueue, rejectedExecutionHandler);

    for (int i = 0; i < 100; i++) {
      
      threadPool.submit(new Callable<Boolean>() {

        @Override
        public Boolean call() throws Exception {
          System.out.println("thread " + String.valueOf(threadNo) + " is called");
          Thread.sleep(10000);
          System.out.println("thread " + String.valueOf(threadNo) + " is awake");
          throw new Exception();
        }

      });
    }
```

代码中定义了大小为10的线程池，for循环提交了20个线程的时候，10个执行线程，10个线程放入了`workingQueue`。当提交到第21个线程的时候，会触发RejectedExecutionHandler。在这里我们配置了CallerRunsPolicy策略。所以会在主线程直接执行该线程。也就是说，在本程序中最多会有11个线程在执行，10个线程在等待。由此限制了线程池的等待线程数与执行线程数