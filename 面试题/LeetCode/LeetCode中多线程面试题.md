## 1. 【简单】[按序打印](https://leetcode-cn.com/problems/print-in-order/)

### 1.1 题目描述

我们提供了一个类：

```java
public class Foo {
  public void first() { print("first"); }
  public void second() { print("second"); }
  public void third() { print("third"); }
}
```

三个不同的线程将会共用一个 Foo 实例。

1. 线程 A 将会调用 first() 方法。

2. 线程 B 将会调用 second() 方法。

3. 线程 C 将会调用 third() 方法。

请设计修改程序，以确保 second() 方法在 first() 方法之后被执行，third() 方法在 second() 方法之后被执行。

**示例 1:**

```
输入: [1,2,3]
输出: "firstsecondthird"
解释: 
有三个线程会被异步启动。
输入 [1,2,3] 表示线程 A 将会调用 first() 方法，线程 B 将会调用 second() 方法，线程 C 将会调用 third() 方法。
正确的输出是 "firstsecondthird"。
```

**示例 2:**

```
输入: [1,3,2]
输出: "firstsecondthird"
解释: 
输入 [1,3,2] 表示线程 A 将会调用 first() 方法，线程 B 将会调用 third() 方法，线程 C 将会调用 second() 方法。
正确的输出是 "firstsecondthird"。
```

**提示：**

- 尽管输入中的数字似乎暗示了顺序，但是我们并不保证线程在操作系统中的调度顺序。
- 你看到的输入格式主要是为了确保测试的全面性。

### 1.2 题解

```java
import java.util.concurrent.CountDownLatch;

class Foo {
    private CountDownLatch second = new CountDownLatch(1);
    private CountDownLatch third = new CountDownLatch(1);
    public Foo() {

    }

    public void first(Runnable printFirst) throws InterruptedException {

        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
        second.countDown();
    }

    public void second(Runnable printSecond) throws InterruptedException {

        second.await();
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
        third.countDown();
    }

    public void third(Runnable printThird) throws InterruptedException {

        third.await();
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
    }
}
```

## 2. 【中等】[交替打印](https://leetcode-cn.com/problems/print-foobar-alternately/)

### 2.1 题目描述

我们提供一个类：

```java
class FooBar {
  public void foo() {
    for (int i = 0; i < n; i++) {
      print("foo");
    }
  }

  public void bar() {
    for (int i = 0; i < n; i++) {
      print("bar");
    }
  }
}
```

两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次。

**示例 1：**

```
输入: n = 1
输出: "foobar"
解释: 这里有两个线程被异步启动。其中一个调用 foo() 方法, 另一个调用 bar() 方法，"foobar" 将被输出一次。
```

**示例 2：**

```
输入: n = 2
输出: "foobarfoobar"
解释: "foobar" 将被输出两次。
```

### 2.2 题解

```java
public class FooBar {
    private int n;
    private Semaphore semaphoreFoo;
    private Semaphore semaphoreBar;

    public FooBar(int n) {
        this.n = n;
        semaphoreFoo = new Semaphore(1);
        semaphoreBar = new Semaphore(0);
        try {
            semaphoreBar.acquire();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            // printFoo.run() outputs "foo". Do not change or remove this line.
            semaphoreFoo.acquire();
            printFoo.run();
            semaphoreBar.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            // printBar.run() outputs "bar". Do not change or remove this line.
            semaphoreBar.acquire();
            printBar.run();
            semaphoreFoo.release();
        }
    }
    
}
```

