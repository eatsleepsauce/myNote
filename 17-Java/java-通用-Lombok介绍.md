#### Lombok简介

Lombok官网：https://projectlombok.org

> # Project Lombok
>
> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
> Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.

Lombok项目是一种自动接通你的编辑器和构建工具的一个Java库。接着，不用再一次写额外的getter或者equals方法。

##### 一、为什么要使用Lombok

Lombok是一款Java开发插件，使得Java开发者可以通过其定义的一些注解来消除业务工程中冗长和繁琐的代码，尤其对于简单的Java模型对象（POJO）。在开发环境中使用Lombok插件后，Java开发人员可以节省出重复构建，诸如getter和setter、构造器、hashCode和equals这样的方法以及各种业务对象模型的accessor和ToString等方法的大量时间。对于这些方法，它能够在编译源代码期间自动帮我们生成这些方法，并没有如反射那样降低程序的性能。

##### 二、如何使用Lombok

Maven 项目使用使用，加入依赖即可：

```xml
<dependencies>
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.16</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

idea工具中需要加入 lombok插件。

##### 三、Lombok原理

Lombok这款插件正是依靠可插件化的Java自定义注解处理API（JSR 269: Pluggable Annotation Processing API）来实现在Javac编译阶段利用 “Annotation Processor” 对自定义的注解进行预处理后生成真正在JVM上面执行的 “Class文件” 。



<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/20180315114050871.png" alt="img" style="zoom: 67%;" />

首先是项目的源代码文件，在经过编译处理以后，lombok会使用自己的抽象语法树去进行注解的匹配，如果在项目中的某一个类中使用了lombok中的注解，那么注解编译器就会自动去匹配项目中的注解对应到在lombok语法树中的注解文件，并经过自动编译匹配来生成对应类中的getter或者setter方法，达到简化代码的目的。


可以看出Annotation Processing是编译器在解析Java源代码和生成Class文件之间的一个步骤。其中Lombok插件具体的执行流程如下：

![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/9033085-529b23d7f309f4d0.png)

从上面的Lombok执行的流程图中可以看出，在Javac 解析成AST抽象语法树之后，Lombok 根据自己编写的注解处理器，动态地修改 AST，增加新的节点（即Lombok自定义注解所需要生成的代码），最终通过分析生成JVM可执行的字节码Class文件。

使用Annotation Processing自定义注解是在编译阶段进行修改，而JDK的反射技术是在运行时动态修改，两者相比，反射虽然更加灵活一些但是带来的性能损耗更加大。

##### 四、常用的Lombok注解

Lombok主要常用的注解有：@Data、@getter、@setter、@NoArgsConstructor、@AllArgsConstructor、@ToString、@EqualsAndHashCode、@Slf4j、@Log4j等。

- @Data注解：在JavaBean或类JavaBean中使用，这个注解包含范围最广，它包含getter、setter、NoArgsConstructor注解，即当使用当前注解时，会自动生成包含的所有方法；

- @getter注解：在JavaBean或类JavaBean中使用，使用此注解会生成对应的getter方法；

- @setter注解：在JavaBean或类JavaBean中使用，使用此注解会生成对应的setter方法；

- @NoArgsConstructor注解：在JavaBean或类JavaBean中使用，使用此注解会生成对应的无参构造方法；

- @AllArgsConstructor注解：在JavaBean或类JavaBean中使用，使用此注解会生成对应的有参构造方法；

- @ToString注解：在JavaBean或类JavaBean中使用，使用此注解会自动重写对应的toStirng方法；

- @EqualsAndHashCode注解：在JavaBean或类JavaBean中使用，使用此注解会自动重写对应的equals方法和hashCode方法；

- @Slf4j：在需要打印日志的类中使用，当项目中使用了slf4j打印日志框架时使用该注解，会简化日志的打印流程，只需调用info方法即可；

- @Log4j：在需要打印日志的类中使用，当项目中使用了log4j打印日志框架时使用该注解，会简化日志的打印流程，只需调用info方法即可；

在使用以上注解需要处理参数时，处理方法如下（以@ToString注解为例，其他注解同@ToString注解）：

@ToString(exclude="column")

排除column列所对应的元素，即在生成toString方法时不包含column参数；

@ToString(exclude={"column1","column2"})

排除多个column列所对应的元素，其中间用英文状态下的逗号进行分割，即在生成toString方法时不包含多个column参数；

@ToString(of="column")

只生成包含column列所对应的元素的参数的toString方法，即在生成toString方法时只包含column参数；

@ToString(of={"column1","column2"})

只生成包含多个column列所对应的元素的参数的toString方法，其中间用英文状态下的逗号进行分割，即在生成toString方法时只包含多个column参数；




