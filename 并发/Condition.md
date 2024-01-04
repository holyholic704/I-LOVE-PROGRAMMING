# Condition

```java
public interface Condition {

    // 当前线程进入等待状态
    void await() throws InterruptedException;

    // 
    void awaitUninterruptibly();

    long awaitNanos(long nanosTimeout) throws InterruptedException;

    boolean await(long time, TimeUnit unit) throws InterruptedException;

    boolean awaitUntil(Date deadline) throws InterruptedException;

    // 唤醒一个等待在Condition上的线程
    void signal();

    // 唤醒所有等待在Condition上的线程
    void signalAll();
}
```

## 参考

