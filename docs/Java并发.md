# Java 多线程

## 单例模式（双重检查锁）

代码实现：

```java
public class Singleton {
    private static volatile Singleton singletonInstance = null;
    private Singleton(){}
    public static Singleton getSingletonInstance() {
        if(singletonInstance==null) {
            synchronized (Singleton.class) {
                if(singletonInstance==null) {
                    singletonInstance = new Singleton();
                }
            }
        }
        return singletonInstance;
    }
}
```

**为什么要用 static 修饰？**

由于是单例模式，因此无法创建对象，获取对象实例时也就不能使用普通方法，必须使用类方法。而类方法中需要访问 singletonInstance，因此该变量也需要定义成 static 类型的。

**为什么要用 volatile 修饰变量**

`singletonInstance = new Singleton();`这段代码执行时分三步：

1. 分配内存空间
2. 初始化 singletonInstance
3. 将 singletonInstance 指向分配的内存地址

由于 JVM 会进行指令重排序，在多线程环境下，可能导致一个线程获得还没有初始化的实例。例如线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

**为什么要锁住类？**

使用 synchronized 关键字进行加锁，确保同一时间只有一个线程可以进入该方法。

如果不加锁，在多线程环境下，当多个线程同时调用 getSingletonInstance() 方法时，可能会出现以下情况：

- 线程 A 进入 if (singletonInstance == null) 判断，此时 singletonInstance 为 null，线程 A 准备创建实例。
- 在线程 A 创建实例之前，线程 B 也进入 if (singletonInstance == null) 判断，由于线程 A singletonInstance 仍然为 null，线程 B 也会创建一个实例。
- 最终会导致创建了多个 Singleton 实例，破坏了单例模式的唯一性。

**为什么要用 double check？**

因为可能存在一个线程判断了第一次 singletonInstance 为 null，此时另外一个线程也判断 singletonInstance 为 null 并创建了实例，第一个线程由于之前判断过了，就会直接创建实例，导致创建了多个实例对象。
