# Lambdas and Streams
<!-- toc -->

##  优先使用 Lambdas 代替匿名类
* 省略所有lambda参数的类型，除非它们的存在使程序更清晰。
* lambdas 没有名字和文档，如果计算的含义不是显而易见的或者超过几行，就不要把它放在alambda中。

## 优先使用方法引用代替 Lambdas

## 优先使用标准的函数式接口

* 优先使用预定义的函数式接口
* 优先使用基本对于基本类型，优先使用基本类型接口，尽可能的避免装箱和拆箱的操作
* 在函数时接口上添加 `@FunctionalInterface` 注解
* 在重载方法时，不要在相同的参数位置重载不同的函数时式接口

| Inteface | Function Signature | Example |
|--|--|--|
| UnaryOperator<T> |T apply(T t) | String::toLowerCase|
| BinaryOperator<T> |T apply(T t1, T t2) | BigInteger::add|
|Predicate<T> | boolean test(T t)| Collection::isEmpty |
|Function<T,R>| R apply(T t)| Arrays::asList|
| Supplier<T>| T get()| Instant::now|
|Consumer<T>| void accept(T t)| System.out::println|

## 明智的使用流

* 使用流时，使用含义明确的 lambda 表达式参数。
* 避免使用流来处理 char 类型的值。  
* 当有意义时再使用流重构现在的代码。
* 适用于流的场景：
    - 统一变化序列中的元素
    - 过滤序列中的元素
    - 使用单一操作组合序列中的元素，例如求和
    - 将元素装入到集合中，同时根据一些属性对元素进行分组、排序等
    - 在元素序列中搜索符满足指定条件的元素

## 在流中使用没有副作用的函数
* forEach 应该只用来输出结果，而不是用于计算。
* 使用静态导入的方式引入 Collectors 的静态成员可以使流更具可读性。

## 优先使用 Collection 而不是 Stream 作为返回值类型
* 在公开的方法中，返回连续的数据序列最好使用 Collection 及其子类。
* 不要为了返回 Collection 类型，而将数据量很大的数据存入内存中。

## 谨慎的使用并行流
* 如果流的数据源来自于 Stream.iterate 或者中间使用了 limit 操作，使用并行流通常无法提高性能。
* 不要轻易的并行化流。
* 由 ArrayList, HashMap, HashSet, and ConcurrentHashMap instances; arrays; intranges; and long ranges 产生的流一般可以使用并行流提高性能
* 并行流不仅有可能影响性能，还有可能影响程序的运行结果。
* 在恰当的环境下，通过 parallel 函数对流进行并行处理可以实现和处理器数量相对应的线性加速比，
