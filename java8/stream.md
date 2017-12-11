# Stream

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
