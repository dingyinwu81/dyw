---
date: 2016-10-25  UTC
title: Java并发之Executor
description: Java1.5提供了一个非常高效实用的多线程包：java.util.concurrent, 提供了大量高级工具，可以帮助开发者编写高效、易维护、结构清晰的Java多线程程序，本文讲的是Executor框架。
permalink: /posts/thread1/
key: 100002
labels: [并发]
encoding: UTF-8
---


Java1.5提供了一个非常高效实用的多线程包：java.util.concurrent, 提供了大量高级工具，可以帮助开发者编写高效、易维护、结构清晰的Java多线程程序，本文讲的是Executor框架。

**Executor**结构如下：
![这里写图片描述](http://img.blog.csdn.net/20161025201952898)


> **1、Runnable与Callable**

（1）Callable规定的方法是**call（）**，Runnable规定的方法是**run（）**。
（2）Callable的任务执行后可返回值，而Runnable的任务是不能返回值得 
（3）call方法可以抛出异常，run方法不可以
（4）运行Callable任务可以拿到一个Future对象，表示异步计算的结果。 它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。 通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。

```
Runnable task = new Runnable() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("run method " + LocalDateTime.now() + "" + Thread.currentThread());
        }
    };

    Callable<String> call = new Callable<String>() {
        @Override
        public String call() throws Exception {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return ("call method " + LocalDateTime.now() + "" + Thread.currentThread());
        }
    };
```


> **2、创建线程池**

**Executors** 类提供了一系列工厂方法用于创线程池，返回的线程池都实现了ExecutorService接口

(1)**newCachedThreadPool**

创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

```
 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
 }
```
(2)**newFixedThreadPool** 

创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
 }
```
(3)**newScheduledThreadPool** 

创建一个定长线程池，支持定时及周期性任务执行。

```
 public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
 }
```
(4)**newSingleThreadExecutor**

创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

```
 public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

```


> **3、ExecutorService 线程池方法**

```
void shutdown();
```
状态则立刻变成SHUTDOWN状态,以后不能再往线程池中添加任何任务，否则将会抛出RejectedExecutionException异常。此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出

```
sList<Runnable> shutdownNow();
```

调用Thread.interrupt来实现线程的立即退出

```
boolean isShutdown();
```
是否executor已经shutdown

```
boolean isTerminated();
```
是否所有的线程都已经运行完成

```
boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
```
直到所有任务都完成后，关闭执行请求，或超时发生，或当前线程是中断，以先发生者为准。

```
<T> Future<T> submit(Callable<T> task);
```


提交一个任务用于执行，返回Future< T>。

```
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
```

执行给定的任务，当所有任务完成时，返回保持任务状态和结果的 Future 列表。

```
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
```

执行给定的任务，如果某个任务已成功完成（也就是未抛出异常），则返回其结果。

下面看具体代码：

```
public class ExecutorDemo1 {

    ExecutorService executor = Executors.newFixedThreadPool(10);

    ExecutorService executorService = Executors.newCachedThreadPool();

    Runnable task = new Runnable() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("run method " + LocalDateTime.now() + "" + Thread.currentThread());
        }
    };

    Callable<String> call = new Callable<String>() {
        @Override
        public String call() throws Exception {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return ("call method " + LocalDateTime.now() + "" + Thread.currentThread());
        }
    };

    @Test   //Runnable
    public void method1() throws Exception {
        for (int i = 0; i < 100; i++) {
            executor.execute(task);
        }
        executor.shutdown();
        executor.awaitTermination(100, TimeUnit.SECONDS);
    }

    @Test   //Callable
    public void method2() throws Exception {
        List<Future<String>> list = new ArrayList<>();
        //1、循环的搭配Future，再添加到List<Future<String>>中
        for (int i = 0; i < 100; i++) {
            Future<String> submit = executor.submit(call);
            list.add(submit);


//            System.out.println(submit.get()); //为什么需要得到List<Future<String>> 再循环，不直接打印呢
        }

        //2、直接的得到List<Callable<String>>，用invokeAll方法得到返回至
//        List<Callable<String>> listCall = new ArrayList<>();
//        for (int i = 0; i < 100; i++) {
//            listCall.add(call);
//        }
//        list = executor.invokeAll(listCall);

        //1 2
        for (Future<String> future : list) {
            System.out.println(future.get());
        }

        executor.shutdown();
        executor.awaitTermination(15, TimeUnit.SECONDS);
    }

    @Test callable 抓取异常
    public void method4() throws Exception {
        List<Future<String>> list = new ArrayList<>();

        for (int i = 0; i < 100; i++) {
            Future<String> submit = executor.submit(new CallableTest(i));
            list.add(submit);
        }
        for (Future<String> future : list) {
            try {
                System.out.println(future.get());
            } catch (InterruptedException e1) {

                e1.printStackTrace();
            } catch (ExecutionException e1) {

                e1.printStackTrace();
            }
        }
        executor.shutdown();
        executor.awaitTermination(15, TimeUnit.SECONDS);
    }

}

//自己抛出异常，自己抓取异常
class CallableTest implements Callable<String> {

    int i;

    CallableTest(int i) {
        this.i = i;
    }

    @Override
    public String call() throws Exception {
        if (i == 50) {
            throw new InterruptedException("我故意抛出的异常");
        }
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return ("call method " + LocalDateTime.now() + "" + Thread.currentThread());
    }
}

```

> **4、CompletionService**

当用Callable接口来实现具有返回值的线程时，使用线程池的submit方法提交Callable任务，利用submit方法返回的Future，调用此存根的get方法来获取整个线程池中所有任务的运行结果。需要用Collection保存所有的Future，然后在主线程中遍历这个Collection并调用 **.get()** 方法取得线程的返回值。这个时候主线程不能保证首先获得的是最先完成的线程返回值，它只是按加入线程池的顺序返回。因为 **.get** 方法是阻塞方法，后面的任务完成了，前面的任务却没有完成，主程序就那样等待在那儿，只到前面的完成了，它才知道原来后面的也完成了。

用**CompletionService**类，它整合了Executor和BlockingQueue（阻塞队列）的功能。你可以将Callable任务提交给它去执行，然后使用类似于队列中的take方法获取线程的返回值。使用CompletionService来维护处理线程不的返回结果时，主线程总是能够拿到最先完成的任务的返回值，而不管它们加入线程池的顺序。

继续接上面的代码：
```
    @Test   //CompletionService
    public void method3() throws Exception {

      CompletionService<String> completionService = new ExecutorCompletionService<String>(executor);

      for (int i = 0; i < 100; i++) {
          completionService.submit(call);
      }

      for(int i = 0; i < 100; i++) {
          String string = completionService.take().get();
          System.out.println(string);
      }

      executor.shutdown();
      executor.awaitTermination(15, TimeUnit.SECONDS);

    }
```


