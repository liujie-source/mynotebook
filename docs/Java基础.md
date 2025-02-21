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

## Java 内部类有哪些？

1. 成员内部类

成员内部类是定义在另一个类的内部，但在方法之外的类。它可以像外部类的成员变量一样，被各种访问修饰符（如 public、private、protected）修饰。

示例代码：

```java
class OuterClass {
    private int outerField = 10;

    // 成员内部类
    public class InnerClass {
        public void display() {
            // 内部类可以访问外部类的私有成员
            System.out.println("Outer field value: " + outerField);
        }
    }

    public void createInnerObject() {
        InnerClass inner = new InnerClass();
        inner.display();
    }
}

public class Main {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.createInnerObject();

        // 也可以直接创建内部类对象
        OuterClass.InnerClass inner = outer.new InnerClass();
        inner.display();
    }
}
```

- 成员内部类 InnerClass 可以访问外部类 OuterClass 的私有成员 outerField。
- 可以在外部类的方法中创建内部类对象，也可以通过外部类对象来创建内部类对象。
- 成员内部类可以使用 OuterClass.this 来引用外部类的实例。

2. 静态内部类

静态内部类是使用 static 关键字修饰的内部类。它不能访问外部类的非静态成员，但可以访问外部类的静态成员。

示例代码：

```java
class OuterClass {
    private static int outerStaticField = 20;

    // 静态内部类
    public static class StaticInnerClass {
        public void display() {
            System.out.println("Outer static field value: " + outerStaticField);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        // 直接创建静态内部类对象，无需创建外部类对象
        OuterClass.StaticInnerClass inner = new OuterClass.StaticInnerClass();
        inner.display();
        OuterClass.StaticInnerClass.display(); //或者直接执行
    }
}
```

- 静态内部类 StaticInnerClass 只能访问外部类的静态成员 outerStaticField。
- 创建静态内部类对象时，不需要先创建外部类对象。

3. 成员内部类

局部内部类是定义在方法或代码块内部的类。它的作用域仅限于定义它的方法或代码块内部。

示例代码：

```java
class OuterClass {
    public void outerMethod() {
        final int localVariable = 30;

        // 局部内部类
        class LocalInnerClass {
            public void display() {
                System.out.println("Local variable value: " + localVariable);
            }
        }

        LocalInnerClass inner = new LocalInnerClass();
        inner.display();
    }
}

public class Main {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.outerMethod();
    }
}
```

- 局部内部类 LocalInnerClass 定义在 outerMethod 方法内部，只能在该方法内部使用。
- 局部内部类可以访问外部方法的 final 局部变量（在 Java 8 及以后，即使没有显式声明为 final，只要局部变量实际上是不可变的，也可以访问）。

4. 匿名内部类

匿名内部类是一种没有名字的局部内部类，通常用于创建一次性的实现类对象。它在创建对象的同时进行类的定义。

示例代码：

```java
interface MyInterface {
    void doSomething();
}

public class Main {
    public static void main(String[] args) {
        // 匿名内部类
        MyInterface obj = new MyInterface() {
            @Override
            public void doSomething() {
                System.out.println("Doing something...");
            }
        };

        obj.doSomething();
    }
}
```

- 匿名内部类实现了 MyInterface 接口，并在创建对象时重写了 doSomething 方法。
- 匿名内部类没有显式的类名，只能使用一次。
