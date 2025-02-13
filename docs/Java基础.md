# Java 基础

## 泛型

### 泛型有哪些限制？

1. 类型擦除：泛型信息在编译后会被擦除，运行时无法获取泛型的实际类型参数。
2. 不能使用基本类型作为类型参数：泛型的类型参数必须是引用类型。
3. 不能创建泛型类型的实例

```java
class MyClass<T> {
    public void createInstance() {
        // 错误：无法创建泛型类型的实例
        T instance = new T();
    }
}
```

```java
class MyClass<T> {
    private Class<T> clazz;

    public MyClass(Class<T> clazz) {
        this.clazz = clazz;
    }

    public T createInstance() throws Exception {
        //正确示例
        return clazz.getDeclaredConstructor().newInstance();
    }
}
```

```java
public class MyClass<T> {
    private T instance = null;
    public T getInstance() {
        //这样是没有问题的
        if(instance==null) instance = new MyClass<T>();
        return instance;
    }
}
```

4. 不能创建泛型数组

```java
public class GenSet<E> {
    private E[] a;
    public GenSet() {
        a = new E[INITIAL_ARRAY_LENGTH]; // 错误：无法创建泛型数组
    }
}
```

使用反射创建泛型数组

```java
import java.lang.reflect.Array;

public class GenSet<E> {
    private E[] a;

    public GenSet(Class<E> c, int s) {
        @SuppressWarnings("unchecked")
        final E[] a = (E[]) Array.newInstance(c, s);
        this.a = a;
    }

    E get(int i) {
        return a[i];
    }
}
```

使用 Object 数组后进行类型转换

```java
public class GenSet<E> {
    private Object[] a;

    public GenSet(int s) {
        a = new Object[s];
    }

    @SuppressWarnings("unchecked")
    E get(int i) {
        return (E) a[i];
    }
}
```

5. 不能使用泛型类型参数进行静态操作：泛型类型参数是与实例相关的，而静态成员是属于类的，与实例无关。

```java
class MyClass<T> {
    // 错误：不能在静态字段中使用泛型类型参数
    private static T value;

    // 错误：不能在静态方法中使用泛型类型参数
    public static void print(T value) {
        System.out.println(value);
    }
}
```

6. 不能抛出或捕获泛型类的实例：Java 的异常处理机制要求异常类型必须是具体的类。

```java
// 错误：不能捕获泛型类的实例
try {
    // ...
} catch (MyGenericException<T> e) { // 错误
    // ...
}
```

7. 不能仅通过泛型类型参数的不同来重载方法

```java
class MyClass {
    // 错误：方法签名冲突
    public void print(List<String> list) {}
    public void print(List<Integer> list) {}
}
```

8. 泛型类型参数不能直接用于继承或实现接口

```java
// 错误：不能使用泛型类型参数作为父类
class MyClass<T> extends T {}

// 错误：不能使用泛型类型参数作为接口
class MyClass<T> implements T {}
```

9. 泛型类型参数不能用于注解

```java
// 错误：不能使用泛型类型参数作为注解
@MyAnnotation
class MyClass<T> {}
```

### 桥方法是什么？

**桥方法（Bridge Method）**是 Java 编译器为了解决泛型类型擦除和多态冲突而自动生成的一种方法。
例如：

```java
class Parent<T> {
    public void set(T value) {
        // ...
    }
}

class Child extends Parent<String> {
    @Override
    public void set(String value) {
        // ...
    }
}
```

在编译后，Parent 和 Child 的 set 方法分别为：

```java
//Parent类
public void set(Object value) {
    // ...
}
//Child类
public void set(String value) {
    // ...
}
```

故而导致 Child 类的 set 方法实际上并没有重写 Parent 类的 set 方法。
因此在编译时会自动生成一段代码：

```java
class Child extends Parent<String> {
    // 子类实际实现的方法
    public void set(String value) {
        System.out.println("Child set: " + value);
    }

    // 编译器生成的桥方法
    public void set(Object value) {
        set((String) value); // 调用子类的 set(String) 方法
    }
}
```

## 集合

### fail-fast 和 fail-safe 是什么？

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
