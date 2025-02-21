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

## wait、sleep、yield 和 join 是否会触发锁的释放以及 CPU 资源的释放？

Object.wait()方法，会释放锁资源以及 CPU 资源。

Thread.sleep()方法，不会释放锁资源，但是会释放 CPU 资源。

yield 会使当前线程重回到可执行状态，等待 cpu 的调度，不释放锁，释放 CPU。

当前线程调用 某线程.join() 时会使当前线程等待某线程执行完毕再结束，底层调用了 wait，释放锁，释放 CPU。

## 线程中断后会释放所占用的锁吗？

当系统停止后，锁将被释放。无论是因为正常退出、异常退出或者强制停止，都会导致锁的释放。这是由 java 虚拟机自动完成的，程序员不需要手动释放锁。

## interrupt 的作用是什么？

interrupt 是 Thread 类的一个实例方法，它的作用是设置调用该方法的线程的中断标志为 true。这个中断标志就像是一个信号，线程可以在合适的时机检查这个标志，并根据情况决定是否响应中断。

不同场景下的表现：

1. 线程处于正常运行状态
   当线程处于正常运行状态（没有处于 sleep、wait、join 等阻塞状态）时，调用 interrupt 方法只会将线程的中断标志设置为 true，线程会继续正常执行，不会立即停止。线程需要在代码中主动检查中断标志来决定是否响应中断。

```java
public class InterruptNormalExample {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            while (true) {
                // 检查中断标志
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程收到中断信号，即将退出");
                    break;
                }
                System.out.println("线程正在正常执行");
            }
        });

        t.start();

        try {
            // 主线程等待一段时间后中断子线程
            Thread.sleep(1000);
            t.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

2. 线程处于阻塞状态
   当线程处于 sleep、wait、join 等阻塞状态时，调用 interrupt 方法会使线程抛出 InterruptedException 异常，同时会清除线程的中断标志（即将中断标志设置为 false）。

3. 使用 lockInterruptibly 方法获取锁时
   当线程使用 Lock 接口的 lockInterruptibly 方法获取锁时，如果在等待锁的过程中被中断，会抛出 InterruptedException 异常。如果线程已经获取到锁，在执行过程中被中断，锁不会自动释放，需要手动调用 unlock 方法。

## notify()会立刻释放锁么？

参考链接：https://www.jianshu.com/p/ffc0c755fd8d >

## SynchronousQueue 介绍

SynchronousQueue 是一种特殊的阻塞队列，它内部并不会存储元素。每一个 put 操作必须等待一个 take 操作，反之亦然，也就是说它是一个元素的直接传递通道，不持有元素。这种特性使得它非常适合用于线程间的直接传递数据，避免了元素在队列中存储的开销。SynchronousQueue 可用作 CachedThreadPool 的工作队列。
