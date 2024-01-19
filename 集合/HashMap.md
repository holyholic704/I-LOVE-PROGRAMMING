# HashMap

## 成员变量

```java
// 默认初始化容量
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 当桶中的元素大于等于该值时，转换成红黑树
static final int TREEIFY_THRESHOLD = 8;

// 当桶中的元素小于等于该值时，转换成链表
static final int UNTREEIFY_THRESHOLD = 6;

// 桶中的元素需转化为红黑树时，table的长度需大于等于该值
static final int MIN_TREEIFY_CAPACITY = 64;

// 存放元素的数组，大小总是2的整数幂
transient Node<K,V>[] table;

// 缓存entrySet()
transient Set<Map.Entry<K,V>> entrySet;

// 元素的数量
transient int size;

// 集合结构的修改次数
transient int modCount;

// 扩容阈值，当前容量*负载因子大于该值时，触发扩容
int threshold;

// 负载因子
final float loadFactor;
```

```java
transient Set<K>        keySet;
transient Collection<V> values;
```

### 元素的数量 size

如果插入元素时没有发生过哈希冲突，那么 HashMap 的元素数量就等于 table 数组中存放的元素的数量

如果插入元素时发生过哈希冲突，那么 HashMap 的元素数量就等于，因哈希冲突而产生的链表与红黑树中节点的数量，加上数组中未发生过哈希冲突的元素数量

### 负载因子 loadFactor

负载因子控制着数组存放数据的疏密程度，因为需要通过 threshold 判断是否需要扩容，而 threshold 是通过 `当前容量 * 负载因子` 计算得来的

- 负载因子越大，扩容阈值也就越大，扩容次数也就越少，数组中元素分布就越紧密，空间利用率高了，但发生冲突的几率也增加了
- 负载因子越小，扩容阈值也就越小，扩容次数也就越多，数组中元素分布就越松散，空间浪费的多了，但发生冲突的几率也减少了

不建议修改 loadFactor，官方推荐的 0.75 就是一个很好的平衡了空间和时间复杂度的值

### 为什么 HashMap 的容量是 2 的倍数

- 索引计算：HashMap 通过 hash 值与容量取余操作来确定 key 的索引位置。当容量为 2 的倍数时，就可将取余操作转化成更高效的位运算，提高计算效率
- HashMap的初始容量是2的次幂，扩容之后的长度是原来的二倍，新的容量也是2的次幂，所以，元素，要么在原位置，要么在原位置再移动2的次幂

## 哈希函数

```java
static final int hash(Object key) {
    int h;
    // 高16位与低16位进行异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

获取到 key 的 hash 值后，再通过 `(n - 1) & hash`，即数组长度减 1 与 hash 进行与运算就可以得到该 key 对应的数组下标位置了

```java
// 例如数组长度为16，hash值为110183（字符串one的哈希值）
0000000000000000000000000001111
0000000000000011010111001100111
// 通过(n - 1) & hash计算出下标值为7
0000000000000000000000000000111
```

`(n - 1) & hash` 等价于 `hash % n`，即对数组长度进行取余

- 位运算相比于取余操作更高效

```java
// 当b为2的n次方时，位运算与取余操作可以相互替换
a % b = a & (b - 1)
// 即
a % 2^n = a & (2^n - 1)
```

2 的整数幂意味着二进制中，只有一位是 1，其他位都是 0，一个数字对 2 的整数幂取余，就意味着该数字向右移 n 位

- 因此 HashMap 的数组长度也都是 2 的整数幂

```java
// 例如110183%16
0000000000000011010111001100111
0000000000000000000000000010000
// 高位补0，低位舍弃
00000000000000000011010111001100111
// 舍弃的部分即为余数
0111
```

2 的整数幂减 1，相当于一个取反操作，例如 `10000` 减 1，也就变成了 `01111`。这时再通过与运算，就可以得到取余操作时舍弃的值了，因为 0、1 与 1 做与运算，会保留原来的值

例如 16 是 2 的 4 次方，在二进制中，就是 1 后面有 4 个 0，对 2 的 4 次方取余，也就是被取余的数向右移 4 位。这时候 16 的二进制 `10000` 作为数字的作用其实已经完成了，可以去协助存储余数了

```java
// 例如110183%16
0000000000000011010111001100111
0000000000000000000000000010000

// 16-1
0000000000000000000000000001111

// 即
0000000000000011010111001100111
0000000000000000000000000001111
// 做与运算，即保留4位
0000000000000000000000000000111
```

### 为什么不直接使用 hash 值做下标

因为不同元素的 hash 值可能是不同的，假如添加两个 key，一个 key 计算出来的 hash 值是 6，那么长度为 16 的数组就足够了，另一个 key 计算出来的 hash 值为 114514，那么至少需要一个长度为 114514 的数组。只存两个元素的话，空间浪费太大了，不如削足适履，就在已有的位置里找

### 为什么需要高 16 位和低 16 位的异或

首先来看看不进行异或操作获取的下标值

```java
// 1 & (16-1)
0000000000000000000000000000001
0000000000000000000000000001111
// 得到
0000000000000000000000000000001

// 65537 & (16-1)
0000000000000010000000000000001
0000000000000000000000000001111
// 得到
0000000000000000000000000000001
```

1 与 65537 对 16 进行取余，得到的结果都是 1。再来看看经过异或运算后

```java
// 1
0000000000000000000000000000001
// 1>>>16
0000000000000000000000000000000
// 1 ^ (1>>>16)
0000000000000000000000000000001
0000000000000000000000000000000
// 得到
0000000000000000000000000000001
// (1 ^ (1>>>16)) & (16-1)
0000000000000000000000000000001
0000000000000000000000000001111
// 得到
0000000000000000000000000000001

// 65537
0000000000000010000000000000001
// 65537>>>16
0000000000000000000000000000001
// 65537 ^ (65537>>>16)
0000000000000010000000000000001
0000000000000000000000000000001
// 得到
0000000000000010000000000000000
// (65537 ^ (65537>>>16)) & (16-1)
0000000000000010000000000000000
0000000000000000000000000001111
// 得到
0000000000000000000000000000000
```

1 与 65537 经过高低 16 位的异或运算，得到了不同的下标，1 对应的下标仍是 1，而 65537 对应的下标变成了 0

通过高低 16 位的异或运算，混合了高位与低位的特征，加大了随机性，且变相的保存了高位的信息，使得数据的分布更松散，降低哈希冲突的概率

- 使用异或运算，而不使用与运算、或运算，是因为异或运算能更好的保留各部分的特征，使用与运算出来的值会向 0 靠拢，使用或运算出来的值会向 1 靠拢

|  | 与 | 或 | 异或 |
| :-: | :-: | :-: | :-: |
| 0 0 | 0 | 0 | 0 |
| 0 1 | 0 | 1 | 1 |
| 1 0 | 0 | 1 | 1 |
| 1 1 | 1 | 1 | 0 |

### 哈希冲突



## 节点

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
}
```

## 构造方法

```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                            initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

## 添加

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```java
// hash：key的hash值
// key：键
// value：值
// onlyIfAbsent：如果是true，不要修改已有的值
// evict：如果是false，
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 如果数组为空，触发扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 通过(n - 1) & hash计算出，该key对应的下标
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

## 扩容

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## 参考

- [二进制按位与&及求余数](https://perkins4j2.github.io/posts/19120/)
- [为什么HashMap使用高16位异或低16位计算Hash值？](https://zhuanlan.zhihu.com/p/458305988)
