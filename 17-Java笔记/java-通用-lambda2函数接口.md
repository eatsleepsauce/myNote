#### Lambda函数接口

##### 一、函数式接口

函数式接口（Functional Interface）就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口，它可以被隐式转换为 Lambda 表达式。

 换句话说函数式接口就是 Lambda 表达式的类型。在函数式接口中，单一方法的命名并不重要，只要方法签名和 Lambda 表达式的类型匹配即可。

函数式接口有下面几个特点：

1. 接口有且仅有一个抽象方法；
2. 允许定义静态方法；
3. 允许定义默认方法；
4. 允许 `java.lang.Object` 中的 `public` 方法；
5. 推荐使用 `@FunctionInterface` 注解（如果一个接口符合函数式接口的定义，加不加该注解都没有影响，但加上该注解可以更好地让编译器进行检查）。

JDK 8 之后新增了一个函数接口包 `java.util.function` 这里面包含了我们常用的一些函数接口：

| 接口           | 参数   | 返回类型 | 说明                                                         |
| -------------- | ------ | -------- | ------------------------------------------------------------ |
| Predicate      | T      | boolean  | 接受一个输入参数 `T`，返回一个布尔值结果                     |
| Supplier       | 无     | T        | 无参数，返回一个结果，结果类型为 `T`                         |
| Consumer       | T      | void     | 代表了接受一个输入参数 `T` 并且无返回的操作                  |
| Function<T,R>  | T      | R        | 接受一个输入参数 `T`，返回一个结果 `R`                       |
| UnaryOperator  | T      | T        | 接受一个参数为类型 `T`,返回值类型也为 `T`                    |
| BinaryOperator | (T, T) | T        | 代表了一个作用于于两个同类型操作符的操作，并且返回了操作符同类型的结果 |

###### 1、Predicate

条件判断并返回一个Boolean值，包含一个抽象方法 (test) 和常见的三种逻辑关系 与 (and) 、或 (or) 、非 (negate) 的默认方法。

```java
import java.util.function.Predicate;

public class DemoPredicate {
    public static void main(String[] args) {
        //条件判断
        doTest(s -> s.length() > 5);
        //逻辑非
        doNegate(s -> s.length() > 5);
        //逻辑与
        boolean isValid = doAnd(s -> s.contains("H"),s-> s.contains("w"));
        System.out.println("逻辑与的结果："+isValid);
        //逻辑或
        isValid = doOr(s -> s.contains("H"),s-> s.contains("w"));
        System.out.println("逻辑或的结果："+isValid);
    }

    private static void doTest(Predicate<String> predicate) {
        boolean veryLong = predicate.test("Hello World");
        System.out.println("字符串长度很长吗：" + veryLong);
    }

    private static boolean doAnd(Predicate<String> resource, Predicate<String> target) {
        boolean isValid = resource.and(target).test("Hello world");
        return isValid;
    }

    private static boolean doOr(Predicate<String> one, Predicate<String> two) {
        boolean isValid = one.and(two).test("Hello world");
        return isValid;
    }
    private static void doNegate(Predicate<String> predicate) {
        boolean veryLong = predicate.negate().test("Hello World");
        System.out.println("字符串长度很长吗：" + veryLong);
    }
}
```

```
字符串长度很长吗：true
字符串长度很长吗：false
逻辑与的结果：true
逻辑或的结果：true
```

###### 2、Supplier

用来获取一个泛型参数指定类型的对象数据（生产一个数据），我们可以把它理解为一个工厂类，用来创建对象。Supplier 接口包含一个抽象方法 `get`，通常我们它来做对象转换。

```java
import java.util.function.Supplier;

public class DemoSupplier {
    public static void main(String[] args) {
        String sA = "Hello ";
        String sB = "World ";
        System.out.println(
                getString(
                        () -> sA + sB
                )
        );
    }

    private static String getString(Supplier<String> stringSupplier) {
        return stringSupplier.get();
    }
}
```

###### 3、Consumer

与 Supplier 接口相反，Consumer 接口用于消费一个数据。Consumer 接口包含一个抽象方法 `accept` 以及默认方法 `andThen` 这样 Consumer 接口可以通过 `andThen` 来进行组合满足我们不同的数据消费需求。

```java
import java.util.function.Consumer;

public class DemoConsumer {
    public static void main(String[] args) {
        //调用默认方法
        consumerString(s -> System.out.println(s));
        //consumer接口的组合
        consumerString(
                // toUpperCase()方法，将字符串转换为大写
                s -> System.out.println(s.toUpperCase()),
                // toLowerCase()方法，将字符串转换为小写
                s -> System.out.println(s.toLowerCase())
        );
    }

    private static void consumerString(Consumer<String> consumer) {
        consumer.accept("Hello");
    }

    private static void consumerString(Consumer<String> first, Consumer<String> second) {
        first.andThen(second).accept("Hello");
    }
}

//Consumer代码
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

输出:

```
Hello
HELLO
hello
```

###### 4、Function

根据一个类型的数据得到另一个类型的数据，换言之，根据输入得到输出。Function 接口有一个抽象方法 `apply` 和默认方法 `andThen`，通过 `andThen` 可以把多个 `Function` 接口进行组合，是适用范围最广的函数接口。

```java
import java.util.function.Function;

public class DemoFunction {
    public static void main(String[] args) {
        doApply(s -> Integer.parseInt(s));
        doCombine(
                str -> Integer.parseInt(str)+10,
                i -> i *= 10
        );
    }

    private static void doApply(Function<String, Integer> function) {
        int num = function.apply("10");
        System.out.println(num + 20);
    }
    private static void doCombine(Function<String, Integer> first, Function<Integer, Integer> second) {
        int num = first.andThen(second).apply("10");
        System.out.println(num + 20);
    }
}

// Function 代码
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before{
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after){
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
   
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

结果：

```
30
220
```

###### 5、UnaryOperator

一元操作

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    /**
     * Returns a unary operator that always returns its input argument.
     *
     * @param <T> the type of the input and output of the operator
     * @return a unary operator that always returns its input argument
     */
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

###### 6、BinaryOperator

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    /**
     * Returns a {@link BinaryOperator} which returns the lesser of two elements
     * according to the specified {@code Comparator}.
     *
     * @param <T> the type of the input arguments of the comparator
     * @param comparator a {@code Comparator} for comparing the two values
     * @return a {@code BinaryOperator} which returns the lesser of its operands,
     *         according to the supplied {@code Comparator}
     * @throws NullPointerException if the argument is null
     */
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    /**
     * Returns a {@link BinaryOperator} which returns the greater of two elements
     * according to the specified {@code Comparator}.
     *
     * @param <T> the type of the input arguments of the comparator
     * @param comparator a {@code Comparator} for comparing the two values
     * @return a {@code BinaryOperator} which returns the greater of its operands,
     *         according to the supplied {@code Comparator}
     * @throws NullPointerException if the argument is null
     */
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);

    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after the function to apply after this function is applied
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     */
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```

