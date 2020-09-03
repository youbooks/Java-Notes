# 如何确保三个线程顺序执行？

场景：有三个线程t1、t2、t3。确保三个线程t1执行完后t2执行，t2执行完成后t3执行。

## 1. 使用join

thread.Join把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的Join\(\)方法，直到线程A执行完毕后，才会继续执行线程B。

> t.join\(\); //调用join方法，等待线程t执行完毕 t.join\(1000\); //等待 t 线程，等待时间是1000毫秒。

```java
public class ThreadTest1 {
// T1、T2、T3三个线程顺序执行
public static void main(String[] args) {
    Thread t1 = new Thread(new Work(null));
    Thread t2 = new Thread(new Work(t1));
    Thread t3 = new Thread(new Work(t2));
    t1.start();
    t2.start();
    t3.start();

}
static class Work implements Runnable {
    private Thread beforeThread;
    public Work(Thread beforeThread) {
        this.beforeThread = beforeThread;
    }
    public void run() {
        if (beforeThread != null) {
            try {
                beforeThread.join();
                System.out.println("thread start:" + Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("thread start:" + Thread.currentThread().getName());
        }
    }
 }
}
```

## 2. 使用CountDownLatch

CountDownLatch\(闭锁\)是一个很有用的工具类，利用它我们可以拦截一个或多个线程使其在某个条件成熟后再执行。它的内部提供了一个计数器，在构造闭锁时必须指定计数器的初始值，且计数器的初始值必须大于0。另外它还提供了一个countDown方法来操作计数器的值，每调用一次countDown方法计数器都会减1，直到计数器的值减为0时就代表条件已成熟，所有因调用await方法而阻塞的线程都会被唤醒。这就是CountDownLatch的内部机制，看起来很简单，无非就是阻塞一部分线程让其在达到某个条件之后再执行。

```java
public class ThreadTest2 {

// T1、T2、T3三个线程顺序执行
public static void main(String[] args) {
    CountDownLatch c0 = new CountDownLatch(0); //计数器为0
    CountDownLatch c1 = new CountDownLatch(1); //计数器为1
    CountDownLatch c2 = new CountDownLatch(1); //计数器为1

    Thread t1 = new Thread(new Work(c0, c1));
    //c0为0，t1可以执行。t1的计数器减1

    Thread t2 = new Thread(new Work(c1, c2));
    //t1的计数器为0时，t2才能执行。t2的计数器c2减1

    Thread t3 = new Thread(new Work(c2, c2));
    //t2的计数器c2为0时，t3才能执行

    t1.start();
    t2.start();
    t3.start();

}

//定义Work线程类，需要传入开始和结束的CountDownLatch参数
static class Work implements Runnable {
    CountDownLatch c1;
    CountDownLatch c2;

    Work(CountDownLatch c1, CountDownLatch c2) {
        super();
        this.c1 = c1;
        this.c2 = c2;
    }

    public void run() {
        try {
            c1.await();//前一线程为0才可以执行
            System.out.println("thread start:" + Thread.currentThread().getName());
            c2.countDown();//本线程计数器减少
        } catch (InterruptedException e) {
        }

    }
 }
}
```

## 3. CachedThreadPool

FutureTask一个可取消的异步计算，FutureTask 实现了Future的基本方法，提空 start cancel 操作，可以查询计算是否已经完成，并且可以获取计算的结果。结果只可以在计算完成之后获取，get方法会阻塞当计算没有完成的时候，一旦计算已经完成，那么计算就不能再次启动或是取消。

一个FutureTask 可以用来包装一个 Callable 或是一个runnable对象。因为FurtureTask实现了Runnable方法，所以一个 FutureTask可以提交\(submit\)给一个Excutor执行\(excution\)。

```java
public class ThreadTest3 {
    // T1、T2、T3三个线程顺序执行
   public static void main(String[] args) {
    FutureTask<Integer> future1= new FutureTask<Integer>(new Work(null));
    Thread t1 = new Thread(future1);

    FutureTask<Integer> future2= new FutureTask<Integer>(new Work(future1));
    Thread t2 = new Thread(future2);

    FutureTask<Integer> future3= new FutureTask<Integer>(new Work(future2));
    Thread t3 = new Thread(future3);

    t1.start();
    t2.start();
    t3.start();
}

 static class Work  implements Callable<Integer> {
    private FutureTask<Integer> beforeFutureTask;
    public Work(FutureTask<Integer> beforeFutureTask) {
        this.beforeFutureTask = beforeFutureTask;
    }
    public Integer call() throws Exception {
        if (beforeFutureTask != null) {
            Integer result = beforeFutureTask.get();//阻塞等待
            System.out.println("thread start:" + Thread.currentThread().getName());
        } else {
            System.out.println("thread start:" + Thread.currentThread().getName());
        }
        return 0;
    }
 } 
}
```

## 4. 使用blockingQueue

阻塞队列 \(BlockingQueue\)是`Java util.concurrent`包下重要的数据结构，BlockingQueue提供了线程安全的队列访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。并发包下很多高级同步类的实现都是基于BlockingQueue实现的。

```java
public class ThreadTest4 {
// T1、T2、T3三个线程顺序执行
public static void main(String[] args) {
    //blockingQueue保证顺序
    BlockingQueue<Thread> blockingQueue = new LinkedBlockingQueue<Thread>();
    Thread t1 = new Thread(new Work());
    Thread t2 = new Thread(new Work());
    Thread t3 = new Thread(new Work());

    blockingQueue.add(t1);
    blockingQueue.add(t2);
    blockingQueue.add(t3);

    for (int i=0;i<3;i++) {
        Thread t = null;
        try {
            t = blockingQueue.take();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t.start();
        //检测线程是否还活着
        while (t.isAlive());
    }
}

static class Work implements Runnable {

    public void run() {
        System.out.println("thread start:" + Thread.currentThread().getName());
    }
 }
}
```

## 5. 使用单个线程池

newSingleThreadExecutor返回以个包含单线程的Executor,将多个任务交给此Exector时，这个线程处理完一个任务后接着处理下一个任务，若该线程出现异常，将会有一个新的线程来替代。

```java
public class ThreadTest5 {

public static void main(String[] args) throws InterruptedException {
    final Thread t1 = new Thread(new Runnable() {
        public void run() {
            System.out.println(Thread.currentThread().getName() + " run 1");
        }
    }, "T1");
    final Thread t2 = new Thread(new Runnable() {
        public void run() {
            System.out.println(Thread.currentThread().getName() + " run 2");
            try {
                t1.join(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }, "T2");
    final Thread t3 = new Thread(new Runnable() {
        public void run() {
            System.out.println(Thread.currentThread().getName() + " run 3");
            try {
                t2.join(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }, "T3");

    //使用 单个任务的线程池来实现。保证线程的依次执行
    ExecutorService executor = Executors.newSingleThreadExecutor();
    executor.submit(t1);
    executor.submit(t2);
    executor.submit(t3);
    executor.shutdown();
 }
}
```

