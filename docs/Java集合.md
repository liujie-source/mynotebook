# Java 集合

## fail-fast 和 fail-safe 是什么？

**fail-fast** 和 **fail-safe** 是 Java 中集合框架的两种不同的迭代器行为机制，主要区别在于它们在遍历集合时对并发修改的处理方式。

1. Fail-Fast（快速失败）

- 当集合在遍历过程中被修改（添加、删除或更新元素），会立即抛出 ConcurrentModificationException 异常。这是 Java 集合框架中大多数集合类的默认行为，例如 ArrayList、HashMap 等。
- 实现原理：
  - 集合内部维护一个 modCount（修改计数器），用于记录集合被修改的次数。
  - 在创建迭代器时，会将 modCount 的值保存到迭代器的 expectedModCount 中。
  - 每次调用迭代器的 next() 或 remove() 方法时，会检查 modCount 是否等于 expectedModCount。如果不相等，说明集合被修改，抛出 ConcurrentModificationException。
- 适用场景：
  - 适用于单线程环境。

2. Fail-Safe（安全失败）

- 允许在遍历集合时对集合进行修改，而不会抛出异常。例如 CopyOnWriteArrayList 和 ConcurrentHashMap。
- 实现原理：
  - 通过创建集合的副本或使用线程安全的集合来实现。
  - 即使原始集合被修改，迭代器也不会受到影响。
- 适用场景：
  - 适用于多线程环境，允许在遍历时对集合进行并发修改。

## HashMap

### HashMap 的容量为什么是 2 的幂?

1. 优化哈希计算

当容量是 2 的幂时，hash(key)%len == hash(key)&(len-1)，而位运算效率更高。

2. 减少哈希冲突，保证哈希值的均匀分布

- 当容量是 2 的幂时，len-1 为全 1，此时索引位置由哈希值的低位部分决定。如果哈希函数设计合理，哈希值的低位部分分布均匀，可以减少哈希冲突的概率。
- 当容量不是 2 的幂时，len-1 中并非全 1，hash(key)&(len-1) 运算后导致某些索引位置永远不会被使用，从而增加哈希冲突的概率。

例如：只有 0 1 8 这三种情况

```
len = 10, len-1 = 1001
hash(key)&(len-1) = 101010 & 1001 = 001000
hash(key)&(len-1) = 100010 & 1001 = 000000
hash(key)&(len-1) = 100001 & 1001 = 000001
```

3. 简化扩容操作，使得扩容机制变得简单和高效

HashMap 在扩容时，容量会变为原来的 2 倍（仍然是 2 的幂），如 111->1111。扩容时，原有的元素需要重新计算索引位置。

- 如果元素的哈希值在扩容后的最高位为 0，则索引不变。
- 如果元素的哈希值在扩容后的最高位为 1，则索引变为 oldIndex + oldLen。

注：这里的最高位指 hash(key) 与新容量的二进制最高位对齐的位置。

### HashMap 多线程操作导致死循环问题

JDK1.7 之前 HashMap 采用头插法插入新的元素，这种插入方式在多线程下扩容时容易导致死循环问题。这是由于当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，头插法可能会导致链表中的节点指向错误的位置，从而形成一个环形链表，进而使得查询元素的操作陷入死循环无法结束。JDK1.8 之后采用尾插法，避免了死循环问题的产生。

HashMap 扩容方法：

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    ......
    //创建一个新的Hash Table
    Entry[] newTable = new Entry[newCapacity];
    //将Old Hash Table上的数据迁移到New Hash Table上
    transfer(newTable);
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}

void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    //下面这段代码的意思是：
    //  从OldTable里摘一个元素出来，然后放到NewTable中
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
```

给定一个长度为 2 的 HashMap，其中有三个元素，现将其扩容为容量为 4 的新的 HashMap。

单线程下 ReHash 过程：

<img src="https://s3.bmp.ovh/imgs/2025/02/13/e1ef59268506e107.jpg" id="self-image"/>

多线程环境下，假设现在有两个线程，第一个线程执行到如下第一条语句，第二个线程已执行完扩容过程。

```java
do {
    Entry<K,V> next = e.next; //Thread 执行到此
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
} while (e != null);
```

<img src="https://s3.bmp.ovh/imgs/2025/02/13/d8362e522f677fca.jpg" id="self-image"/>

线程 1 被调度回来继续执行扩容操作，插入 key(3)：

<img src="https://s3.bmp.ovh/imgs/2025/02/13/18d5bee11c08c291.jpg" id="self-image"/>

插入 key(7)：

<img src="https://s3.bmp.ovh/imgs/2025/02/13/11710c9ec76d1151.jpg" id="self-image"/>

继续插入 key(3)，最终导致链表循环：

<img src="https://s3.bmp.ovh/imgs/2025/02/13/eed2bef699dea9b3.jpg" id="self-image"/>

参考链接：https://coolshell.cn/articles/9606.html >

## ConcurrentHashMap

### ConcurrentHashMap 是如何加入元素的？

Java 8 中 ConcurrentHashMap 加入元素的过程：

1. 计算哈希值

- 使用键的 hashCode 计算哈希值，并通过扰动函数进一步散列，以减少哈希冲突。

2. 定位桶

- 根据哈希值找到对应的桶（数组索引）。

3. 插入元素

- 如果桶为空，使用 CAS 操作将新节点插入。
- 如果桶不为空，则锁住当前桶（使用 synchronized），然后遍历链表或红黑树：
  - 如果找到相同的键，则更新值。
  - 如果没有找到相同的键，则将新节点插入链表或红黑树。

4. 检查扩容

- 插入完成后，检查是否需要扩容。如果需要扩容，则启动扩容操作。

### ConcurrentHashMap 在扩容时可以查找和添加元素吗？

可以。

- 查找操作（读操作）：

  - 如果查找的键值对尚未迁移到新数组，则从旧数组中查找。
  - 如果查找的键值对已经迁移到新数组，则从新数组中查找。

- 添加操作（写操作）：
  - 如果插入的桶尚未迁移，可以直接插入到旧数组中。
  - 如果插入的桶正在迁移，线程会协助完成迁移，然后再插入数据。

## PriorityQueue
