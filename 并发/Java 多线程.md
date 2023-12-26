### 多线程

为什么要使用多线程

线程安全

死锁：如何预防和避免线程死锁

实现方法

Thread 方法

sleep() 方法和 wait() 方法对比

为什么 wait() 方法不定义在 Thread 中

可以直接调用 Thread 类的 run 方法吗

反复调用同一个线程的start()方法是否可行

假如一个线程执行完毕（此时处于TERMINATED状态），再次调用这个线程的start()方法是否可行

守护线程

线程组

线程的优先级

线程间有哪些通信

父子线程怎么共享数据

## Java 线程的状态

Java 中线程被分成了 6 种状态

```java
public enum State {

    // 新生状态
    NEW,

    // 运行状态，也可能是等待状态
    RUNNABLE,

    // 阻塞状态
    BLOCKED,

    // 等待状态
    WAITING,

    // 超时等待状态
    TIMED_WAITING,

    // 终止状态
    TERMINATED;
}
```
