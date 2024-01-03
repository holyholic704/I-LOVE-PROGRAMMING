# AQS

AQS 是 AbstractQueuedSynchronizer 的简称，即抽象队列同步器，是一个用来构建锁和同步器的框架，Java 中众多的锁都是基于 AQS 构建的

![](./md.assets/aos.png)

AQS 继承自 AOS（AbstractOwnableSynchronizer），该类主要用于表示锁与持有者的关系（独占模式）

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    private static final long serialVersionUID = 3737899427754241961L;

    protected AbstractOwnableSynchronizer() { }

    // 锁的持有者
    private transient Thread exclusiveOwnerThread;

    // 设置锁持有者
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }

    // 获得锁持有者
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

- AQS 定义了两种资源共享方式：Exclusive（独占，只有一个线程能执行）和 Share（共享，多个线程可同时执行）

AOS 有两个子类 AQS 与 AQLS（AbstractQueuedLongSynchronizer），在 AQS 中资源是用一个 int 类型的数据表示的，当业务需要更大范围的资源时，可以使用 AQLS，他们俩的代码几乎一致，只是 AQLS 将资源的表示类型改为了 long 类型

```java
// AQS
private volatile int state;

// AQLS
private volatile long state;
```

## AQS 的结构

AQS 是基于模版方法模式的抽象类，需要子类去实现一些方法，并且多以内部类的形式实现

```java
// 以ReentrantLock为例，内部类Sync就是继承自AQS
public class ReentrantLock implements Lock, java.io.Serializable {

    abstract static class Sync extends AbstractQueuedSynchronizer {
        ...
    }
    ...
}
```

AQS 虽然是抽象类，但内部并没有抽象方法，而是定义了一些只抛出异常没有具体实现的 protected 方法，提供给子类去实现。这样做的好处是避免子类将所有的抽象方法都实现一遍，只需要实现自己需要的方法即可

```java
// 独占模式，尝试获取资源
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}

// 独占模式，尝试是否资源
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}

// 共享模式，尝试获取资源
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}

// 共享模式，尝试释放资源
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}

// 当前线程是否正在独占资源
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
}
```

## 原理

AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是基于 CLH 锁 （Craig, Landin, and Hagersten locks） 实现的

![](./md.assets/clh.png)

<small>[AQS 详解 - CLH 队列](https://javaguide.cn/java/concurrent/aqs.html#aqs-%E6%A0%B8%E5%BF%83%E6%80%9D%E6%83%B3)</small>

CLH 锁是对自旋锁的一种改进，是一个虚拟的双向队列（不存在队列实例，仅存在结点之间的关联关系），暂时获取不到锁的线程将被加入到该队列中。AQS 将每条请求共享资源的线程封装成一个 CLH 队列锁的一个结点（Node）来实现锁的分配

- 每个节点会不断轮询前驱节点的状态，如果发现前驱节点释放了锁就结束自旋，自旋超过一定次数仍未获取到锁，会将该线程阻塞
- head 指向已获得锁的节点，且自身不持有具体线程

```java
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

```java
static final class Node {
    // 标记一个节点在共享模式下进行等待，用作nextWaiter的取值之一
    static final Node SHARED = new Node();
    // 标记一个节点在独占模式下进行等待，用作nextWaiter的取值之一
    static final Node EXCLUSIVE = null;
    // 取消状态
    static final int CANCELLED =  1;
    // 唤醒状态
    static final int SIGNAL    = -1;
    // 条件等待状态
    static final int CONDITION = -2;
    // 传播状态
    static final int PROPAGATE = -3;

    // 当前节点的等待状态，初始值为0，可在以上4个状态中切换
    volatile int waitStatus;
    // 前驱节点
    volatile Node prev;
    // 后继节点
    volatile Node next;
    // 当前节点持有的线程
    volatile Thread thread;
    // 下一个等待节点，指的其实也就是自己的等待模式
    Node nextWaiter;

    // 当前节点是否处于共享模式
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    // 获取前驱节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {
    }

    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}

// 等待队列的头节点
private transient volatile Node head;

// 等待队列的尾节点
private transient volatile Node tail;
```

```java
// 根据传入的模式创建节点并加入到等待队列中
// 接受两种模式，Node.EXCLUSIVE代表独占模式, Node.SHARED代表共享模式
private Node addWaiter(Node mode) {
    // 从调用的构造方法能看出，Node内部类中nextWaiter指的就是该节点的等待模式
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

AQS 使用 state 表示同步状态，通过内置的等待队列来完成获取资源线程的排队工作。是所有想获取该锁的线程共享的

```java
// 同步状态
private volatile int state;

// 获取同步状态
protected final int getState() {
    return state;
}

// 设置同步状态
protected final void setState(int newState) {
    state = newState;
}

// 通过CAS操作设置同步状态
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

state 的初始值为 0，表示锁处于未锁定状态，所有线程每次能成功获取到锁就加 1

- 独占模式下同一线程多次获得同一个锁也是会累加的，通过在方法中对当前请求的线程与锁持有者做对比，相同才会累加，不同则抛出异常

### 获取资源

- 共享模式获取资源

```java
public final void acquireShared(int arg) {
    // 判断条件由子类进行具体实现
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 自旋
        for (;;) {
            final Node p = node.predecessor();
            // 如果当前节点的前驱节点是头结点，表示当前节点可以尝试获取资源了
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 设置头结点，并且传播获取资源成功的状态，这个方法的作用是确保唤醒状态传播到所有的后继节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            // 判断获取资源失败是否需要阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

- 独占模式获取资源

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 自旋
        for (;;) {
            final Node p = node.predecessor();
            // 如果当前节点的前驱节点是头结点，表示当前节点可以尝试获取资源了
            if (p == head && tryAcquire(arg)) {
                // 设置头结点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 判断获取资源失败是否需要阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

共享模式与独占模式获取资源的代码很类似，区别在于共享模式同步器当节点获取资源成功晋升为头节点之后，它会把自身的等待状态通过 CAS 更新为 Node.PROPAGATE，下一个加入等待队列的新节点会把头节点的等待状态值更新回 Node.SIGNAL

简单来说共享模式获取资源成功后，就会通知之后的节点顶上，而独占模式获取资源成功后，因为是独占的，所以后面的还需要再等等

## 参考

- [AQS 详解](https://javaguide.cn/java/concurrent/aqs.html)
- [源码分析:AQS源码](https://blog.csdn.net/maoyeqiu/article/details/95866862)
