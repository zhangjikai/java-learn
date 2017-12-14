# Stream

<!-- toc -->

## 定义
流 （Stream）允许我们以声明性方式（declarative way）处理数据集合。流的简短定义为：a sequence of elements from a source that supports data processing operations（从支持数据处理操作的源生成的元素序列），我们来详细看下这个定义：
* Sequence of elements（元素序列）- 流提供了类似于集合（Collection）的接口，额可以操作特定元素类型的一组有序值。不过两者的关注点不同，集合主要关注数据（data），流主要关注计算（computation）。
* Source（源）-  流需要一个提供数据的源，这个源可以是 collections，arrays 或者 I/O resources. 从有序集合生成的流会保留原有的数据。
* Data processing operations（数据处理操作）- 流的数据处理功能支持类似于数据库的操作，以及函数式编程语言中的常用操作，例如 filter、map、reduce、find、match、sort 等。流操作可以顺序执行，也可以并行执行。

另外，流操作还有两个重要的特点：
* Pipelining（流水线） - 很多流操作本身会返回一个流，这样多个操作就可以链接起来，形成一个大的流水线，流水线的操作支持延迟（laziness）和短路（short-circuiting）。流水线的操作可以看作对数据源进行数据库式的查询。
* Internal iteration（内部迭代）- 和集合使用显式的的迭代器不同，流的迭代操作是在背后执行的，对于用户来说是透明的。

下面是一个使用示例：
```java
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class HelloWorld {

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(5);
        list.add(10);
        list.add(1);
        list.add(-1);
        list.add(1000);
        list.add(100);

        List<Integer> result = list.stream()
            .filter(i -> i > 10)
            .sorted()
            .collect(Collectors.toList());

        result.forEach(System.out::println);
    }
}
```
## 使用流

### 创建流
下面是创建流的几种方式：
* 由值创建流
* 由数组 / 集合创建流
* 由文件生成流
* 由函数生成流，该操作生成的流属于无限流
  - `Stream.iterate`：接受一个初始值，每次迭代在之前迭代的结果上应用传入 lambda 表达式
    ```java
    public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)`
    ```
  - `Stream.generate`：根据传入的 s 生成结果值
    ```java
    public static<T> Stream<T> generate(Supplier<T> s)
    ```

下面是一个使用示例：
```java
import org.junit.Test;

import java.io.IOException;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class CreateStream {

    @Test
    public void fromValue() {
        Stream<String> stream = Stream.of("aaa", "bbb", "ccc", "ddd");
        stream.map(String::toUpperCase).forEach(System.out::println);
    }

    @Test
    public void fromCollection() {
        List<String> list = new ArrayList<>();
        list.add("111");
        list.add("222");
        list.add("333");
        Integer sum = list.stream()
            .map(Integer::parseInt)
            .reduce(Integer::sum).get();
        System.out.println(sum);
    }

    @Test
    public void fromArray() {
        int[] arr = new int[]{1, 3, 5, 7};
        IntStream stream = Arrays.stream(arr);
        int sum = stream.sum();
        System.out.println(sum);

        String[] strArr = new String[]{"aaa", "bbb", "ccc", "ddd"};
        Stream<String> stringStream = Arrays.stream(strArr);
        stringStream.map(String::toUpperCase).forEach(System.out::println);
    }

    @Test
    public void fromFile() {
        long uniqueWords = 0;
        try (Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
            uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))).distinct().count();
            System.out.println(uniqueWords);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 由函数生成流
     */
    @Test
    public void fromIterate() {
        Stream.iterate(0, n -> n + 2)
            .limit(10)
            .forEach(System.out::println);
    }

    @Test
    public void fromGenerate() {
        Stream.generate(Math::random)
            .limit(5)
            .forEach(System.out::println);
    }
}
```
<br />
### 只能遍历一次
流只能被遍历一次，当遍历完之后，我们就说这个流被消费掉了。下面是一个示例：
```java
@Test
public void streamTest() {
    Stream<Integer> stream = Stream.of(1, 3, 5);
    stream.filter(i -> i > 1).forEach(System.out::println);
    // 下面的代码会报错，因为流已经关闭了
    stream.filter(i -> i > 1).forEach(System.out::println);
}
```

### 流操作
流操作可以分为两大类：中间操作（intermediate operations）和终端操作（terminal operation）。中间操作是延迟（lazy）计算的，在触发一个终端操作之前，中间操作不会进行任何处理。下面是一个示意图：

![](/images/stream/operation.png)

中间操作一般可以合并起来，在终端操作时一次性全部处理。在使用流时一般会包含下面三个部分：
* 执行查询的 **数据源（data source）**
* 组成流水线的 **中间操作链（chain of intermediate operations）**
* 执行流水线并且生成最终结果的 **终端操作（terminal operation）**

下面是常用的流操作列表

| 操作 | 操作类型 | 返回类型 | 参数类型 | 函数描述符 |
| --- | --- | --- | --- | --- | --- |
| `filter`| 中间 | `Stream<T>` | `Predicate<T>` | `T -> boolean` |
| `distinct` | 中间 <br>有状态-无界 | `Stream<T>` | | |
| `skip`| 中间 <br>有状态-有界| `Stream<T>` | long | |
| `limit` | 中间<br>有状态-有界 | `Stream<T>` | | |
| `map` | 中间 | `Stream<R>` | `Function<T, R>` | `T -> R` |
| `flatMap`| 中间 | `Stream<R>` | `Function<T, Stream<R>>` | `T -> Stream<R>` |
| `sorted` | 中间 | `Stream<T>` | `Comparator<T>` | `(T, T) -> int`|
| `anyMatch` | 终端 | `boolean` | `Predicate<T>` | `T -> boolean` |
| `noneMatch` | 终端 | `boolean` | `Predicate<T>` | `T -> boolean` |
| `allMatch` | 终端 | `boolean` | `Predicate<T>` | `T -> boolean` |
| `findAny` | 终端 | `Optional<T>` | | |
| `findFirst` | 终端 | `Optional<T>` | | |
| `forEach` | 终端 | `void` | `Comsumer<T>`| `T -> void` |
| `collect` | 终端 | `R` | `Collector<T, A, R>` | |
| `reduce` | 终端 <br> 有状态-有界| `Optional<T>` | `BinaryOperator<T>` | `(T, T) -> T` |
| `count` | 终端 | `long` | | | |

下面是部分流操作的示例
```java
public void streamOperations() {
    Integer[] arr = new Integer[]{1, 2, 6, 1, 3, 8, 10};

    // filter: 根据条件过滤
    // forEach: 遍历流中的元素
    Arrays.stream(arr).filter(i -> i > 1).forEach(System.out::println);
    System.out.println();

    // distinct: 去重
    Arrays.stream(arr).distinct().forEach(System.out::println);
    System.out.println();

    // sorted: 排序
    Arrays.stream(arr).sorted().forEach(System.out::println);
    System.out.println();

    // skip: 跳过前面的元素
    Arrays.stream(arr).skip(2).forEach(System.out::println);
    System.out.println();

    // limit: 限制获取的元素数量
    Arrays.stream(arr).limit(2).forEach(System.out::println);
    System.out.println();

    // map: 元素映射， T -> R
    Arrays.stream(arr).map(String::valueOf).forEach(v -> System.out.println(v.length()));
    System.out.println();

    // anyMatch:任一元素匹配
    boolean result = Arrays.stream(arr).anyMatch(i -> i > 5);
    System.out.println(result);

    // noneMatch: 无元素匹配
    result = Arrays.stream(arr).noneMatch(i -> i > 5);
    System.out.println(result);

    // allMatch: 全部元素匹配
    result = Arrays.stream(arr).allMatch(i -> i > 5);
    System.out.println(result);

    // findAny: 返回流中任一元素
    Optional<Integer> optional = Arrays.stream(arr).findAny();
    System.out.println(optional.get());

    // findFirst: 返回流中的第一个元素
    optional = Arrays.stream(arr).findFirst();
    System.out.println(optional.get());

    // count: 统计流中元素个数
    long count = Arrays.stream(arr).count();
    System.out.println(count);
}
```
### flatMap
map 主要的的功能是用来做映射，即将输入的 T 类型转为 R 类型。map 主要针对一维的数据结构（例如一维数组），当数据结构为二维的时候（比如二维数组或者集合数组），map 也会将其作为一维数据结构处理，比如下面的二维数组，map 会将内层的一维数组看作是一个整体。
```
[
  [1, 3, 4],
  [1, 2, 3],
  [2, 2, 6]
]
```
但是我们可能希望对二维数组中的全部数据进行处理，那么一个简单的办法就是将二维数组展平为一维数组，即下面的形式：
```
[1, 3, 4, 1, 2, 3, 2, 2, 6]
```
这样在 map 时就是针对的数组中的全部数据。Java 8 为我们提供了 flatMap 来完成上面的工作。我们首先看下 flatMap 的定义：
```java
/**
 * Returns a stream consisting of the results of replacing each element of
 * this stream with the contents of a mapped stream produced by applying
 * the provided mapping function to each element.  Each mapped stream is
 * {@link java.util.stream.BaseStream#close() closed} after its contents
 * have been placed into this stream.  (If a mapped stream is {@code null}
 * an empty stream is used, instead.)
 */
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```
我们看到 flatMap 接受的参数为 `Function<? super T, ? extends Stream<? extends R>`，这个 Function 的功能是将 T 转换为 `Stream<R>` (super 和 extends 一会再说)。flatMap 的功能就是通过转换函数 mapper，将之前流中的每个元素转换成一个流，然后把所有的流连接成一个流。下面是一个示例：
```java
public void useFlatMap() {
    Integer[][] array = new Integer[][]{
        new Integer[]{1, 3, 4},
        new Integer[]{1, 2, 3},
        new Integer[]{2, 2, 6}
    };

    List<Integer[]> results = Arrays.stream(array).distinct().collect(Collectors.toList());
    results.forEach(arr -> System.out.println(Arrays.toString(arr)));

    List<Integer> results2 = Arrays.stream(array).flatMap(Arrays::stream).distinct().collect(Collectors.toList());
    results2.forEach(System.out::println);
}
```
在上面的例子中，通过 Arrays.stream 可以将二维数组 array 转换为一个流 s，其中流中的元素数据类型为 `Integer[]`，当调用 flatMap 之后， 流 s 中的每个元素都通过 `Arrays.stream` 方法转换为一个流，记为 ss1, ss2, ss3，流 ss1, ss2, ss3 中数据类型为 `Integer`，然后 flatMap 将 ss1, ss2 和 ss3 合并成一个新的流 s2，流 s2 中就包含了 array 的所有元素（相当于把 array 展平为了一维数组）。

 下面是一个示意图：

![](/images/stream/flatmap.png)

我们再来看下泛型中的 super 和 extends，
```java
// 这里只看泛型，所以省略 ? extends Stream
Function<? super T, Stream<? extends R> mapper
```

`? super T` 表示接受的类型是 T 或者 T 的父类，`? extends R` 表示接受的类型为 R 或者 R 的子类。假设我们有下面的继承关系：
```java
class Animal {
    public void run() {
        System.out.println("animal run");
    }
}

class Cat extends Animal {
    public void jump() {
        System.out.println("cat jump");
    }
}

class WhiteCat extends Animal {
    public void printColor() {
        System.out.println("white");
    }
}
```

我们先来看 super，假设 Stream 中元素类型都是 Cat 类型，即 mapper 的输入类型是 Cat，假设我们不加 `? super T` 限制，将参数的定义改为 `Function mapper`, 那么将下面的 Function m 传入 flatMap 时，编译时不会报错，但是执行的时候会报错，因为 Function m 接受的类型是 WhiteCat，但是我们给 m 传的 Cat 类型，Cat 类中根本没有 printColor 函数。
```java
// 这里忽略输出类型
Function<WhiteCat, ?> m = (c, ?) -> c.printColor();
```
如果我们将下面的 Function m2 传入 flatMap，那么编译和运行时都不会报错，因为 Cat 是 Animal 的子类，会继承 Animal 的 run 方法。
```java
Function<Animal, ?> m2 = (a, ?) -> a.run();
```
由此我们可以知道当 Stream 中类型为 Cat 时，我们传入的 Function 接受类型可以是 Cat 或者 Cat 的父类，但不能是 Cat 的子类，所以在参数定义时加上 `? super T`，以便在编译阶段就可以发现错误。

下面我们在看下 extends，mapper 的用法大致如下面的代码所示，流 s 中的数据类型要为 R 或者 R 的子类，所以在参数定义中使用 `? extends R` 表明 mapper 返回流中的数据必须为 R 或者 R 的子类。
```java
Stream<R> s = mapper(t);
```
上面的用法就是我们所说的 PECS （Producer Extends Consumer Super）元素：
* 如果要从对象或者集合中获取 T 类型的数据，并且不需要写入，可以使用 `? extends` 通配符
* 如果要向对象或者集合中写入 T 类型的数据，并且不需要读取，可以使用 `? super` 通配符
* 如果既要存又要取，那么就不要用任何通配符。

在上面的例子中，我们要向 mapper 对象中传入 T 类型，获取 `Stream<R>` 类型，所以统配符使用
```java
Function<? super T, ? extends Stream<? extends R>> mapper
```

### reduce
reduce 操作将流中所有的值规约成一个值（比如求和、求最大最小值），用函数式编程语言的术语来说，这也称为折叠（fold）。下面是一个使用示例：
```java
public void useReduce() {
    Integer[] numbers = new Integer[] {1, 2, 4, 6, 10};

    // 求和
    int sum = Arrays.stream(numbers).reduce(0, Integer::sum);
    System.out.println(sum);

    // 求最大值
    Optional<Integer> o = Arrays.stream(numbers).reduce(Integer::max);
    System.out.println(o.get());

    // 求最小值
    o = Arrays.stream(numbers).reduce(Integer::min);
    System.out.println(o.get());
}
```
reduce 将外部迭代的过程抽象到了内部迭代中，有利于并行化的实现。

### 操作状态
* 对于 map 或者 filter 这样的流操作来说，它们操作的对象是流中的单个元素，在执行操作的时候，各个元素之间不会相互影响，这些操作一般都是无状态的（stateless），这里假设用户提供的 Lambda 表达式中没有内部状态
* 诸如 reduce、sum、max 等操作，需要内部状态来累积结果，但是这种情况下内部状态很小，在 reduce 中就是一个int 或者 double。不论流中有多少元素要处理，内部状态都是有界的。
* 对于 sort 或者 distinct 操作来说，操作时都需要知道之前操作的结果，这种操作的存储要求是无界的，我们通常把这些操作称为有状态操作（stateful）
