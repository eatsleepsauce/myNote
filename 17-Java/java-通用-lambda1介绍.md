#### Lambda介绍

##### 一、Lambda表达式介绍

###### 1、什么是Lambda表达式

>Lambda expression in computer programming, also called an anonymous function, is a defined function not bound to an identifier. ——维基百科

 Lambda 表达式也叫作匿名函数，是一种是未绑定标识符的函数定义，在编程语言中，匿名函数通常被称为 Lambda 抽象。换句话说， Lambda 表达式通常是在需要一个函数，但是又不想费神去命名一个函数的场合下使用。这种匿名函数，在 JDK 8 之前是通过 Java 的匿名内部类来实现，从 Java 8 开始则引入了 Lambda 表达式 —— 一种紧凑的、传递行为的方式。

###### 2、Lambda表达式的优点

- **更加紧凑的代码**： Lambda 表达式可以通过省去冗余代码来减少我们的代码量，增加我们代码的可读性；

- **更好地支持多核处理**： Java 8 中通过 Lambda 表达式可以很方便地并行操作大集合，充分发挥多核 CPU 的潜能，并行处理函数如 filter、map 和 reduce；

- **改善我们的集合操作**： Java 8 引入 Stream API，可以将大数据处理常用的 map、reduce、filter 这样的基本函数式编程的概念与 Java 集合结合起来。方便我们进行大数据处理。

###### 3、Lambda怎么发挥多核处理？





![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f1a899609c28f5805810286.jpg)



##### 二、Lambda表达式语法

###### 1、基本语法

```java
button.addActionListener(event -> System.out.println("button click"));
```

最基本的 Lambda 表达式，它由三部分组成具体格式是这样子的：

```
参数 -> 具体实现
```

包含一个 Lambda 表达式的运算符 `->`，在运算符的左边是输入参数，右边则是函数主体。(一段带有输入参数的可执行语句块)。

Lambda 表达式有这么几个特点：

- **可选类型声明：** 不需要声明参数类型，编译器可以自动识别参数类型和参数值。

- **可选的参数圆括号：** 一个参数可以不用定义圆括号，但多个参数需要定义圆括号；

- **可选的大括号：** 如果函数主体只包含一个语句，就不需要使用大括号；

- **可选的返回关键字：** 如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指明表达式返回了一个数值。

###### 2、Lambda表达式常见的形式

（1）**不包含参数**

```java
Runnable noArguments = () -> System.out.println("hello world");
```

Runnable 接口，只有一个 run 方法，没有参数，且返回类型为 void，所以我们的 Lambda 表达式使用 () 表示没有输入参数。

（2）**有且只有一个参数**

```java
ActionListener oneArguments = event -> System.out.println("hello world");
```

（3）**有多个参数**

```
BinaryOperator<Long> add = (x,y) -> x+y ;
```

（4）**表达式主体是一个代码块**

```java
Runnable noArguments = () -> {
    System.out.print("hello");
    System.out.println("world");
}
```

（5）**显示声明参数类型**

```java
BinaryOperator<Long> add = (Long x, Long y) -> x+y ;
```

###### 3、Lambda表达式的参数类型

```java
BinaryOperator<Long> add = (x,y) -> x+y ;
```

参数 `x`，`y` 和返回值 `x+y` 我们都没有指定具体的类型，但是编译器却知道它是什么类型。原因就在于编译器可以从程序的上下文推断出来，这里的上下文包含下面 3 种情况：

- 赋值上下文；
- 方法调用上下文；
- 类型转换上下文。

通过这 3 种上下文就可以推断出 Lambda 表达式的目标类型。

![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f1a8c99099b657309380350.jpg)

##### 三、Lambda变量和作用域

###### 1、访问局部变量

Lambda 表达式不会从父类中继承任何变量名，也不会引入一个新的作用域。Lambda 表达式 **基于词法作用域**，也就是说 Lambda 表达式函数体里面的变量和它外部环境的变量具有相同的语义。

访问局部变量要注意如下 3 点：

**（1）可以直接在 Lambda 表达式中访问外层的局部变量**

在 Lambda 表达式中可以直接访问外层的局部变量，但是这个局部变量必须是声明为 `final` 的。

```java
import java.util.function.BinaryOperator;

public class LambdaTest1 {
    public static void main(String[] args) {
        final int delta = -1;
        BinaryOperator<Integer> add = (x, y) -> x+y+delta;
        Integer apply = add.apply(1, 2);//结果是2
        System.out.println(apply);
    }
}
```

 `delta` 是 Lambda 表达式中的外层局部变量，被声明为 `final`，我们的 Lambda 表达式是对两个输入参数 `x`,`y` 和外层局部变量 `delta` 进行求和。

如果这个变量是一个既成事实上的 final 变量的话，就可以不使用 `final` 关键字。所谓个既成事实上的 final 变量是指只能给变量赋值一次，`delta` 只在初始化的时候被赋值，所以它是一个既成事实的 `final` 变量。

**（2） 在 Lambda 表达式当中被引用的变量的值不可以被更改**

```java
public static void main(String...s){
    int delta = -1;
    BinaryOperator<Integer> add = (x, y) -> x+y+ delta; //编译报错
    add.apply(1,2);  
    delta = 2;
}
```

```
Variable used in lambda expression should be final or effectively final
```

**（3）在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量**

```java
public static void main(String...s){
    int delta = -1;
    BinaryOperator<Integer> add = (delta, y) -> delta + y + delta; //编译报错
    add.apply(1,2);  
}
```

```
Variable 'delta' is already defined in the scope
```

###### 2、访问对象字段与静态变量

**Lambda 内部对于实例的字段和静态变量是即可读又可写的。**

```java
import java.util.function.BinaryOperator;

public class Test {
    public static int staticNum;
    private int num;

    public void doTest() {
        BinaryOperator<Integer> add1 = (x, y) -> {
            num = 3;
            staticNum = 4;
            return x + y + num + Test.staticNum;
        };
        Integer apply = add1.apply(1, 2);
        System.out.println(apply);
    }

    public static void main(String[] args) {
        new Test().doTest();
    }
}
```

###### 3、引用值，而不是变量

 Lambda 表达式可以读写实例变量，只能读取局部变量。

实例变量和局部变量的实现不同：实例变量都存储在堆中，而局部变量则保存在栈上。如果在线程中要直接访问一个非`final`局部变量，可能线程执行时这个局部变量已经被销毁了。因此，Java 在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就没有这个限制（也就是既成事实的 `final`）。



![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f1a8ac809552cdc06870210.jpg)

##### 四、Lambda表达式的引用

所谓 Lambda 表达式的方法引用可以理解为 Lambda 表达式的一种快捷写法，相较于通常的 Lambda 表达式而言有着更高的**可读性**和**重用性**。

一般而言，方法实现比较简单、复用地方不多的时候推荐使用通常的 Lambda 表达式，否则应尽量使用方法引用。

Lambda 表达式的引用分为：**方法引用** 和 **构造器引用**两类。

###### 1、方法引用

方法引用的格式为：

```
类名::方法名
```

通常引用语法格式有以下 3 种：

（1）**静态方法引用**

所谓静态方法应用就是调用类的静态方法。

- 被引用的参数列表和接口中的方法参数一致；

- 接口方法没有返回值的，引用方法可以有返回值也可以没有；

- 接口方法有返回值的，引用方法必须有相同类型的返回值。

```java
//创建一个带有静态方法的类
public class StaticMethodClass{
    public static int doFind(String s1, String s2){
        return s1.lastIndexOf(s2);
    }
}

// 接口
public interface Finder {
    public int find(String s1, String s2);
}

// 定义 Finder 接口引用了 StaticMethodClass 的静态方法 doFind。
Finder finder = StaticMethodClass :: doFind;
```



（2）**参数方法引用** （比较常用）

参数方法引用顾名思义就是可以将 **参数的一个方法** 引用到 Lambda 表达式中。

```java
public interface Finder {
    public int find(String s1, String s2);
}

// 正常Lambda写法
Finder finder =(s1，s2)-> s1.indexOf(s2);

//参数方法引用
Finder finder = String :: indexOf;

//调用find方法
int findIndex = finder.find("abc","bc")
//输出find结果。
System.out.println("返回结果:"+findIndex)
```



（3）**实列方法引用** 

实例方法引用就是直接调用实例的方法，接口方法和实例的方法必须有相同的参数和返回值。

```java
// 接口
public interface Serializer {
    public int serialize(String v1);
}

public class StringConverter {
    public int convertToInt(String v1){
        return Integer.valueOf(v1);
    }
}

StringConverter stringConverter = new StringConverter(); 
Serializer serializer = stringConverter::convertToInt;
```



###### 2、构造器引用

构造器引用的格式为：

```
类名::new
```

构造器引用便是引用一个类的构造函数，接口方法和对象构造函数的参数必须相同。

```java
public interfact MyFactory{
    public String create(char[] chars)
}

MyFactory myfactory =  String::new;
//等价于
MyFactory myfactory = chars->new String(chars);
```



![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f1a8e19094aabd107810143.jpg)

##### 五、Lambda 和 匿名函数

###### 1、无标识性，Lambda表达式，产生了新对象么？

内部类会确保创建一个拥有唯一表示的对象，而 Lambda 表达式的计算结果有可能有唯一标识，也可能没有。

在 Lambda 表达式出现之前，一个 Java 程序的行为总是与对象关联，以标识、状态和性为为特征。然而 Lambda 表达式则违背了这个规则。

虽然 Lambda 表达式可以共享对象的一些属性，但是 **表示行为 是其唯一的用处**。由于没有状态，所以表示问题也就不那么重要了。在 Java 语言的规范中对 Lambda 表达式  **唯一的要求就是必须计算出其实现的相当的函数接口的实例类**。

如果 Java 对每个 Lambda 表达式都拥有唯一的表示，那么 Java 就没有足够的灵活性来对系统进行优化。







###### 2、作用域规则

由于内部类可以从父类继承属性，Lambda 表达式却不能，所以，内部类的作用域规则比 Lambda 表达式要复杂。

匿名内部类与大多数类一样，由于它可以引用从父类继承下来的名字，以及声明在外部类中的名字，所以它的作用域规则非常复杂。

Lambda 表达式由于不会从父类型中继承名字，所以它的作用于规则要简单很多。除了参数以外，用在 Lambda 表达式中的名字的含义与表达式外面是一样的。



```java
public class TestLambdaAndInnerClass  {
    public void test(){
        //匿名类实现
        Runnable innerRunnable = new Runnable(){
            @Override
            public void run() {
                System.out.println("call run in innerRunnable:\t"+this.getClass());
            }
        };
        //Lambda表达式实现
        Runnable lambdaRunnable = () -> System.out.println("call run in lambdaRunnable:\t"+this.getClass());
        new Thread(innerRunnable).start();
        new Thread(lambdaRunnable).start();
    }

    public static void main(String...s){
        new TestLambdaAndInnerClass().test();
    }
}
```

结果：

```
call run in innerRunnable:  class com.github.x19990416.item.TestLambdaAndInnerClass$1  
call run in lambdaRunnable: class com.github.x19990416.item.TestLambdaAndInnerClass
```

Lambda 表达式的 `this` 指针指向的是其外围类 `TestLambdaAndInnerClass`，匿名内部类的指针指向的是其本身。（对于 `super` 也是一样的结果）

**匿名类的 `this`、`super` 指针指向的是其自身的实例，而 Lambda 表达式的 `this`、`super` 指针指向的是创建这个 Lambda 表达式的类对象的实例。**

对于 Lambda 表达式而言，编译器并不认为它是一个完全的类（或者说它是一个特殊的类对象），所以也不具备一个完全类的特征。

![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f1a901009e3945909000228.jpg)

