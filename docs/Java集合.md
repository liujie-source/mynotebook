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

<img src="https://s3.bmp.ovh/imgs/2025/02/13/e1ef59268506e107.jpg" id="self-image"/>
<img src="https://s3.bmp.ovh/imgs/2025/02/13/d8362e522f677fca.jpg" id="self-image"/>
<img src="https://s3.bmp.ovh/imgs/2025/02/13/18d5bee11c08c291.jpg" id="self-image"/>
<img src="https://s3.bmp.ovh/imgs/2025/02/13/11710c9ec76d1151.jpg" id="self-image"/>
<img src="https://s3.bmp.ovh/imgs/2025/02/13/eed2bef699dea9b3.jpg" id="self-image"/>

参考链接：https://coolshell.cn/articles/9606.html >
