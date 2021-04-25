#### Lambda应用

##### 一、函数式数据处理

Java 8 中新增的特性其目的是为了帮助开发人员写出更好地代码，其中关键的一部分就是对核心类库的改进。流（ Stream ）和集合类库便是核心类库改进的内容。

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.stream.Stream;

public class Test{
    public static void main(String...s){
        List<Integer> numbers = new ArrayList<Integer>();
        Collections.addAll(numbers,new Integer[]{1,2,3,4,5,6,7});
        Stream stream1 = numbers.stream();
        numbers.remove(6);
        //直接使用numbers的stream()
        long counter = numbers.stream().filter(e->e>5).count();
        System.out.println(counter);
        //调用之前的stream1
        counter = stream1.filter(ex-> (Integer)ex>5).count();
        System.out.println(counter);
    }
}
```

输出：

```
1
1
```

取到 Stream 对象 stream1 后删除了数组 `numbers` 中的最后一个元素，随后分别对 numbers 和 stream1 进行过滤统计操作，会发现两个结果是一样的，stream1 中的内容跟随 `numbers` 一起做相应的改变。这说明 **Stream 对象不是一个新的集合，而是创建新集合的配方**。同样，像 `filter()` 虽然返回 Stream 对象，但也只是对 Stream 的刻画，并没有产生新的集合。

对于这种不产生集合的方法叫做 **惰性求值方法**，相对应的类似于 `count()` 这种返回结果的方法我们叫做 **及早求值方法**。

我们可以把多个惰性求值方法组合起来形成一个惰性求值链，最后通过及早求值操作返回想要的结果。这类似建造者模式，使用一系列操作设置属性和配置，最后通过一个 `build` 的方法来创建对象。通过这样的一个过程我们可以让我们对集合的构建过程一目了然。

##### 二、常用的流操作

###### 1、collect

`collect` 操作是根据 Stream 里面的值生成一个列表，它是一个求值操作。

`collect` 方法通常会结合 `Collectors` 接口一起使用，是一个通用的强大结构，可以满足数据转换、数据分块、字符串处理等操作。

（1）**生成集合**

使用 `collect(Collectors.toList())` 方法从 Stream 中生成一个列表。

```java
import java.util.stream.Stream;
import java.util.List;
import java.util.stream.Collectors;

public class Test{
	public static void main(String...s){  
		List<String> collected = Stream.of("a","b","c").collect(Collectors.toList());  
		System.out.println(collected);  
	}
}

输出：[a, b, c]
```

（2）**集合转换**

使用 `toCollection` 来定制集合收集元素，这样就把 `List` 集合转换成了 `TreeSet`

```java
import java.util.List;
import java.util.stream.Stream;
import java.util.stream.Collectors;
import java.util.TreeSet;

public class Test{
	public static void main(String...s){
		List<String> collected = Stream.of("a","b","c","c").collect(Collectors.toList());
		TreeSet<String> treeSet = collected.stream().collect(Collectors.toCollection(TreeSet::new));
		System.out.println(collected);
		System.out.println(treeSet);
	}
}

输出结果： 
[a, b, c, c]
[a, b, c]
```

（3）**转换成值**

使用 `maxBy` 接口让收集器生成一个值，通过方法引用调用了 `String` 的 `compareTo` 方法来比较元素的大小。同样还可以使用 `minBy` 来获取最小值。

```java
import java.util.List;
import java.util.stream.Stream;
import java.util.stream.Collectors;

public class Test{
	public static void main(String...s){
		List<String> collected = Stream.of("a","b","c").collect(Collectors.toList());
		String maxChar = collected.stream().collect(Collectors.maxBy(String::compareTo)).get();
		System.out.println(maxChar);
	}
}

输出： c
```

（4）**数据分块**

通过 `partitioningBy` 接口可以把数据分成两类，即满足条件的和不满足条件的，最后将其收集成为一个 `Map` 对象，其 `Key` 为 `Boolean` 类型，`Value` 为相应的集合元素。

同样我们还可以使用 `groupingBy` 方法来对数据进行分组收集，这类似于 SQL 中的 `group by` 操作。

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Stream;
import java.util.stream.Collectors;

public class Test{
	public static void main(String...s){
		List<Integer> collected = Stream.of(1,2,3,4,5,6,7).collect(Collectors.toList());
		Map<Boolean,List<Integer>> divided = collected.stream().collect(Collectors.partitioningBy(e -> e>5));
		System.out.println(divided.get(true));
		System.out.println(divided.get(false));
	}
}

输出结果：
[6, 7]
[1, 2, 3, 4, 5]
```

（5）**字符串处理**

我们把 `collected` 数组的每个元素拼接起来，并用 `[` `]` 包裹。

```java
import java.util.List;
import java.util.stream.Stream;
import java.util.stream.Collectors;

public class Test{
public static void main(String...s){
    List<String> collected = Stream.of("a","b","c").collect(Collectors.toList());
    String formatted = collected.stream().collect(Collectors.joining(",","[","]"));
    System.out.println(formatted);
}
}

输出：[a,b,c]
```

###### 2、map

`map` 操作是将流中的对象换成一个新的流对象，是 Stream 上常用操作之一。 其示意图如下：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f03f03508ff160305730426.jpg" alt="图片描述" style="zoom: 67%;" />

```java
import java.util.List;
import java.util.stream.Stream;
import java.util.stream.Collectors;

public class Test{
	public static void main(String...s){
	    List<String> collected = Stream.of("a","b","c").collect(Collectors.toList());
	    List<String> upperCaseList = collected.stream().map(e->e.toUpperCase()).collect(Collectors.toList());
	    System.out.println(upperCaseList);    
	}
}
```

###### 3、flatmap

`flatmap` 与 `map` 功能类似，只不过 `map` 对应的是一个流，而 `flatmap` 可以对应多个流。

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;

public class Test{
	public static void main(String...s){
	    List<String> nameA = Arrays.asList("Mahela", "Sanga", "Dilshan");
	    List<String> nameB = Arrays.asList("Misbah", "Afridi", "Shehzad");
	    List<List<String>> nameSets = Arrays.asList(nameA,nameB);
	    List<String> flatMapList = nameSets.stream()
	            .flatMap(pList -> pList.stream())
	            .collect(Collectors.toList());
	    System.out.println(flatMapList);
	}
}

返回结果： [Mahela, Sanga, Dilshan, Misbah, Afridi, Shehzad]
```

###### 4、filter

`filter` 用来过滤元素，在元素遍历时，可以使用 `filter` 来提取我们想要的内容，这也是集合常用的方法之一。其示意图如下：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/5f03f08e08b3ddae05220441.jpg" alt="图片描述" style="zoom:67%;" />

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Collections;
import java.util.stream.Collectors;


public class Test{
	public static void main(String...s) {
	    List<Integer> numbers = new ArrayList<>();
	    Collections.addAll(numbers,new Integer[]{1,2,3,4,5,6,7});
	    List<Integer> collected = numbers.stream()
									    .filter(e->e>5).collect(Collectors.toList());
	    System.out.println(collected);
	}
}

输出：[6, 7]
```

###### 5、max/min

`max/min` 求最大值和最小值也是集合上常用的操作。它通常会与 `Comparator` 接口一起使用来比较元素的大小。示例如下：

```java
import java.util.List;
import java.util.Collections;

public class Test{
	public static void main(String...s) {
	    List<Integer> numbers = new ArrayList<>();
	    Collections.addAll(numbers,new Integer[]{1,2,3,4,5,6,7});
	    Integer max = numbers.stream().max(Comparator.comparing(k->k)).get();
	    Integer min = numbers.stream().min(Comparator.comparing(k->k)).get();
	    System.out.println("max:"+max);
	    System.out.println("min:"+min);
	}
}
输出：
max:7
min:1
```

###### 6、reduce

`reduce` 操作是可以实现从流中生成一个值，我们前面提到的如 `count`、`max`、`min` 这种及早求值就是由`reduce` 提供的。

```java
import java.util.stream.Stream;

public class Test{
	public static void main(String...s) {
	    int sum = Stream.of(1,2,3,4,5,6,7).reduce(0,(acc,e)-> acc + e);
	    System.out.println(sum);
	}
}

输出：28
```

例子是对数组元素进行求和，这个时候我们就要使用 `reduce` 方法。这个方法，接收两个参数，第一个参数相当于是一个初始值，第二参数则为具体的业务逻辑。 上面的例子中，我们给 `acc` 参数赋予一个初始值 0 ，随后将 `acc` 参数与各元素求和。



