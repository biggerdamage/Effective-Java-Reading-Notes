# Chapter 7 Lambda表达式和流
本章内容都是Java 8新增特性的使用相关.

## 第42条 优先使用lambdas而不是匿名类
以前, 只有一个抽象方法的接口(或抽象类)被当做function types使用. 它们的实例是函数对象(function objects), 表示功能或者行为.
从JDK1.1开始, 主要的创建函数对象的行为是匿名类(anonymous class).

匿名类很适合经典的需要函数对象的设计模式, 比如策略模式. 举例 -> `Comparator`提供了抽象策略, 匿名类实现具体策略.

在Java 8中, 认为这种只有一个抽象方法的接口值得被特殊对待, 它们现在被称为函数式接口(functional interfaces), 语言允许你用lambda表达式创建这些接口的实例.

Lambda和匿名类的功能类似, 但是更加简洁.

Lambda表达式可以省略参数和返回值的类型, 这是因为编译器会通过类型推断(type inference)推导出来. (不能推导出来的时候需要指明.)

举例: 之前各个枚举行为不同的例子, 可以用lambda简化, 构造的时候传入lambda, 用`DoubleBinaryOperator`保存.

注意: 由于lambda是没有名字和文档的, 如果一个计算不是自解释的, 或是行数较多(对于lambda来说, 一行最好, 三行最多), 就不要放在lambda中了.

通过enum构造传入的参数是在静态环境的, 所以从enum构造传入的lambda不能访问枚举的成员变量.

仍然有一些事情是匿名类可以做, 但是不能被lambda取代的: 
* lambda只被限制在函数式接口(functional interfaces), 所以抽象类, 多个方法的接口仍然要用匿名类.
* 在lambda中, 不能获取自身的引用, `this`关键字指的是enclosing instance. 匿名类中`this`指这个匿名类对象.

lambda和匿名类都不能可靠地序列化和反序列化, 所以, 你应该少做这件事. 如果真的有需要, 用一个私有静态内部类的实例.

## 第43条 优先使用方法引用而不是lambdas
用lambda取代匿名类的主要优势就是简洁, 其实Java还提供了更简洁的生成函数对象的方法: 方法引用(method references).

代码实例: 统计单词个数的方法, 用Map接口的`merge`方法(Java 8):
```
map.merge(key, 1, (count, incr) -> count + incr);
```
改进:
```
map.merge(key, 1, Integer::sum);
```

为了更加简洁, 可以把lambda中的代码提取到一个新方法中, 然后用这个方法引用取代原来的lambda.
但是偶然的情况下, lambda比方法引用更加简洁. 比如: 方法和lambda在同一个类中.

很多方法引用都是静态的, 但是有四种非静态的: 
* bound and unbound instance method references
* classes和arrays的constructor references, 作为工厂对象.

结论: 当方法引用比lambda更加简洁的时候, 就用方法引用, 否则就用lambda.

## 第44条 优先使用标准的函数式接口
有了lambda之后, 模板方法(Template Method)模式就没有吸引力了, 现代的方法是提供一个接收函数对象的静态工厂或者构造函数来达到相同的效果.
更一般地, 你需要写更多的以函数对象作为参数的构造器和方法. 要谨慎选择正确的函数参数类型.

`java.util.function`包中提供了一系列标准的函数式接口(一共43个).

六个基本的函数式接口:
* `UnaryOperator<T>`: 一个参数, 返回值类型和参数相同.
* `BinaryOperator<T>`: 两个参数, 返回值类型和参数相同.
* `Predicate<T>`: 一个参数, 返回一个boolean.
* `Function<T,R>`: 参数和返回值类型不同.
* `Supplier<T>`: 无参数, 有返回值.
* `Consumer<T>`: 有参数, 无返回值.

每个基本接口都有3种变型, 对应基本类型: int, long和double, 接口名字会加上类型的前缀. (6*3=18个)

`Function`接口还有9种变型, 对应结果的不同基本类型. 
如果源和结果都是基本类型, 加上前缀`SrcToResult`, 比如`LongToIntFunction`. (6种).
如果源是基本类型, 结果是对象引用, 加上前缀`SrcToObj`, 比如`DoubleToObjFunction`. (3种).

有三个基本的函数式接口都有双参数版本, 共有9种变型:
* `BiPredicate<T, U>` (1种).
* `BiFunction<T, U, R>`: 有三种返回不同基本类型的变种. (4种).
* `BiConsumer<T>`: 还有接受一个基本类型和一个对象引用的变种. (4种).

最后还有`BooleanSupplier`是一个返回布尔值的Supplier变型.

大多数的标准函数式接口仅是为了提供基本类型(primitive types)支持而存在. 
不要用基本函数式接口搭配装箱基本类型(basic functional interfaces with boxed primitives)来代替基本类型的函数式接口(primitive functional interfaces).

那什么时候应该写自己的函数式接口呢?
* 没有标准的函数式接口能够满足需求. -> 比如参数个数不同, 或者要抛出一个受检异常.
* 有结构相同的标准函数式接口, 有一些情况也应该写自己的函数式接口. 
比如`Comparator<T>`和`ToIntBiFunction<T,T>`结构相同, 但是仍然用`Comparator`, 好处: 名字; 通用协议; 有用的默认方法.

如果你要自己写一个函数式接口而不是用标准的, 你要考虑它是不是和`Comparator`一样拥有(一个或多个)以下特性:
* 它通用, 会得益于有一个描述性的名字.
* 它与一个很强的协议相关.
* 它会得益于自定义的默认方法.

永远记住用注解`@FunctionalInterface`来标记你的函数式接口.

提供方法重载的时候要注意, 不要给同一个方法提供函数式接口在同一个参数位置的重载(有可能会引起二义性). 比如: `ExecutorService`的`submit`方法.
这需要客户端代码进行强转来指明正确的重载.

## 第45条 谨慎使用streams
### Stream API介绍
Java 8新增的streams API主要是为了更方便地进行批量操作, 串行的或者并行的.
两个关键的抽象: 
* stream: 有限或无限的数据元素序列. (支持对象引用或基本类型: int, long, double).
* stream pipeline: 对这些元素进行的多级运算. (0个或多个intermediate operations和一个terminal operation).

stream pipeline的运算是lazy的, 只有当terminal operation被触发的时候才会开始进行运算, 对最终操作不必要的数据将不会被运算(无限数据成为可能).

streams API是流式的.

### 为什么要谨慎使用Streams
适当地使用streams API可以让程序更简洁, 但是使用不当(过度使用)可能会降低可读性和可维护性.

举例: 寻找anagram(相同字母异序词).
Java 8新增方法: `computeIfAbsent`.

关于stream pipeline的可读性:
* 缺少明确的类型时, lambda参数的良好命名是必要的.
* 使用辅助方法, 提取逻辑并命名.

不要用stream来处理char values, 因为它们是int值, 需要强转.

### Stream和循环迭代的比较
stream pipeline使用函数对象(function objects), 通常是lambda和方法引用; 循环迭代(iterative code)使用的是代码块.

代码块可以做但是函数对象不能做的事情(循环可以做, 但stream不可以做):
* 代码块中可以读取和修改scope中的任何局部变量; lambda中只能读取final的, 不能修改任何局部变量.
* 循坏代码块中可以return, break或continue, 抛出方法声明的受检异常; lambda中不能做这些.

stream擅长的事情:
* 统一处理元素序列.
* 过滤.
* 联合元素的运算(加, 连接, 算最小值等).
* 将元素序列累积到集合中, 或分组.
* 查询.

用stream的时候比较难做到的一点是, 在pipeline的处理过程中, 很难找到对应的原来元素: 一旦map到新的值了, 前面的值就丢了.
比较lame的做法是用个pair, 还可以考虑在需要原始值的时候反向map.

例子: 打印20个梅森质数(Mersenne primes): 2的n次方仍然是个质数. 现在想找出对应的n -> 加一个反向运算.

有时候迭代循环和stream都能很好地解决问题(例: 遍历扑克牌组合), 具体选哪一种看情况(更喜欢哪个就用哪个, 同事是否接受你的偏好等).

## 第46条 优先使用streams中没有副作用的方法
Streams不仅是API, 它是基于函数式编程的饿一个范式.

Streams范式中最重要的一点: 把计算组织成一系列的转换, 每一个阶段都尽量是前一个阶段结果的单纯函数(pure function: 无状态 -> free of side-effects).
注意: pure function的结果只依赖于自己的输入, 不会依赖于任何mutable的状态, 也不会修改任何状态.

举例: 统计单词频次: `forEach` -> bad; `collect` -> good.

Stream中的`forEach`有着浓烈的迭代气息, 应该只被用来报告结果, 不应该用来进行计算, 通常是bad smell.

```
freq.keySet().stream() 
.sorted(comparing(freq::get).reversed()) 
.limit(10)
.collect(toList()); // 静态引入了Collectors
```

Collectors API主要是封装了降维操作(reduction strategy), 它的结果通常是一个集合.

注: Scanner的`stream`方法是Java 9加的, 之前的版本可以用`streamOf(Iterable<E>)`.

Collectors API中的方法(一共有39个):
* `toList()`, `toSet()`, `toCollection(collectionFactory)`: 把stream中的元素放在一个集合中.
* `toMap(keyMapper, valueMapper)`: 参数的两个方法, 一个把元素转换到key, 一个转换到value. -> 冲突的时候会抛异常.
* `toMap(keyMapper, valueMapper, mergeFunction)`: 同样key的value会merge. 应用1: 选择value, 例: 选择歌手最畅销的唱片; 应用2: 最后的value胜出.
* `toMap(keyMapper, valueMapper, mergeFunction, mapSupplier)`: mapSupplier是一个map工厂, 用来指定具体的map实现: EnumMap或TreeMap.
* 三个`toMap`方法都有相应的`toConcurrentMap`方法, 创建`ConcurrentHashMap`实例.
* `groupingBy(classifier)`方法用来返回按照类别分组的元素, 分类器(classifier function)返回元素应属于的类别(category), 作为map的key.
* `groupingBy(classifier, downstream)`: downstream将对分类中的所有元素进一步处理, 可以`toSet()`, `toCollection()`或者`counting()`.
* `groupingBy(classifier, mapFactory, downstream)`: 可以同时控制map和集合的实现类型, 比如一个`TreeMap`, 包含`TreeSet`.
* 类似的, 三个`groupingBy`方法的重载都提供`groupingByConcurrent`方法.
* `partitioningBy(predicate)`返回key为`Boolean`类型的map, 还有一种重载形式加了downstream参数.
* 仅用作downstream的方法. 比如`counting`, 以`summing`, `averaging`, `summarizing`开头的方法, `Stream`中都有相应的方法. 还有`reducing`, `filtering`, `mapping`, `flatMapping`, `collectingAndThen`等.
* `minBy`和`maxBy`是`BinaryOperator`中相应方法的类似物.
* `joining`: 有三种重载, 以特定分隔符和前后缀组合字符串.


注: 文档: [Java 8 Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html).

## 第47条 优先使用Collection而不是Stream作为返回值
`Stream`虽然有一个符合`Iterable`接口的规定的用于遍历的方法, 但是`Stream`却没有继承`Interable`接口. 
所以想要遍历Stream, 需要提供一个适配方法:
```
// Adapter from  Stream<E> to Iterable<E>
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
       return stream::iterator;
}
```

如果想要用stream pipeline处理一个Iterable的接口, 也需要写个适配方法:
```
// Adapter from Iterable<E> to Stream<E>
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
       return StreamSupport.stream(iterable.spliterator(), false);
}
```

如果你在写一个方法, 返回一系列的对象, 你应该为使用者要用stream pipeline和for-each的两种可能性都做好准备.

`Collection`接口是`Iterable`的子类型, 还有一个`stream`方法, 所以`Collection`或其一个合适的子类型, 通常是返回序列的公有方法返回值的最好选择.

数组有`Arrays.asList`和`Stream.of`方法.

但是不要把很大的序列放在内存中, 仅仅是为了作为集合返回. -> 举例: PowerSet. -> 自定义collection.
自定义集合可以限制元素大小, 但如果数据太大或者数量不确定, 可以考虑用stream或者iterable.

有时候也会根据实现的方法更直观而选择返回类型, 例子: 用stream处理生成字符串子集集合.

PS: 如果以后`Stream`继承了`Iterable`, 就可以自由返回stream了, 因为这样就两种可能性(流式, 遍历)都提供了.

## 第48条 把流变为并行时要多加小心
如果一个pipeline的source是从Stream.iterate来的, 或者`limit`作为方法在其中被使用了, 那么把这个pipeline变为并行是不太可能提高性能的.
举例: 寻找20个梅森素数的例子, 加上`parallel()`方法之后程序不输出了, 只能强制停止.

不要不加选择地把stream并行化, 性能影响可能是灾难性的.

### 数据结构
并行的性能增益在这些流上是最好的: `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap`实例; 数组; int ranges; long ranges.
这些数据结构的的共同特点:
* 它们都可以准确且廉价地分成任何所需大小的子集, 这使得在并行线程之间划分工作变得容易.
* 它们都提供了优秀的引用地址(locality of reference): 顺序元素引用在内存中存储在一起. 
引用地址(Locality-of-reference)对于并行化批量操作至关重要. 没有它, 线程将会大部分时间空闲, 等待数据从内存传输到处理器缓存中.

### stream pipeline的最终操作也会影响并行的执行.
如果比起整个pipeline的工作来说, 最终操作承担了大量的工作, 而且这个操作还是串行的, 那么把pipeline并行化的作用很有限.
对并行来说, 最好的最终操作是降维操作(reductions), 比如`reduce`, `max`, `min`, `count`, `sum`.
短路操作比如`anyMatch`, `allMatch`, `nonMatch`对于并行来说也适用.
但是Stream的`collect`方法(mutable reductions), 就不是很适合并行了, 因为合并集合开销大.


如果你自己实现`Stream`, `Iterable`或者`Collection`, 想要比较好的并行性能, 你必须覆写`spliterator`方法, 并且广泛地测试流的并行性能.

并行化流不仅会导致性能不佳，包括活动失败(liveness failures); 它可能导致不正确的结果和不可预测的行为(safety failures).

在正确的情形下, 在stream pipeline上加上`parallel`调用是有可能实现处理器内核数量的近线性加速的.

如果要平行的流是随机数字的, 应该使用`SplittableRandom`, 而不是`ThreadLocalRandom`(单线程用), 或`Random`(每个操作都同步).