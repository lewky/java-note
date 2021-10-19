# JDK8 常用库

## `Optional`

```java
Optional<String> opt = Optional.empty(); // 空值
// opt = Optional.of(null); // of()的参数必须不为null
opt = Optional.ofNullable(null); // ofNullable()的参数可为null或非null

// 访问Optional中的值
// System.out.println(opt.get()); // 空值为报NoSuchElementException
System.out.println(opt.isPresent());
// opt = Optional.of("fjh");
// opt.ifPresent(System.out::println);

// Optional中的值不为空值时，返回默认值
// orElse()与orElseGet()的区别：空值时都返回默认值；不为空值时都返回该值，但orElse()会执行逻辑，而orElseGet()不会执行逻辑
System.out.println(opt.orElse("default value"));
System.out.println(opt.orElseGet(() -> "default value"));

// 返回异常
// opt.orElseThrow(NullPointerException::new);

// 转换Optional中的值
opt.map(String::toUpperCase).orElse("default value");

// 过滤值
// opt = Optional.of("abc");
System.out.println(opt.filter(s -> s.contains("a")).orElse("not a"));
```

## `java.time`

```java
// LocalDate LocalTime LocalDateTime 的互相转换
LocalDateTime localDateTime = LocalDate.now().atTime(LocalTime.now());
LocalDate localDate = localDateTime.now().toLocalDate();
LocalTime localTime = localDateTime.now().toLocalTime();

// 日期时间运算
System.out.println(localDateTime.plusDays(1));
System.out.println(localDate.plusMonths(1));
System.out.println(localTime.minusHours(6));

// 与字符串的转换
// LocalDateTime -> String
System.out.println(localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
// String -> LocalDateTime
System.out.println(
        LocalDateTime.parse("2000-01-01 23:59:11", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
System.out.println(
        LocalDateTime.from(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").parse("2001-01-01 23:39:39")));

// Date LocalDateTime的互相转换
// Date -> LocalDateTime
System.out.println(LocalDateTime.ofInstant(new Date().toInstant(), ZoneId.of("Asia/Shanghai")));
// LocalDateTime -> Date
System.out.println(Date.from(LocalDateTime.now().toInstant(ZoneOffset.of("+8"))));
```

## `stream api`

将要处理的元素集合看作一种流，流在管道中传输，并且可以在管道的节点上进行处理(筛选、排序、聚合)，流在管道中经过中间操作(`intermediate operation`)的处理，最后由最终操作(`terminal operation`)得到最终处理的结果

1. `stream`并不是某种数据结构，它只是数据源的一种视图
2. `stream`的任何修改都不会修改背后的数据源
3. `stream`上的操作并不会立即执行
4. `stream`只能遍历一次，想再次遍历必须重新生成

### 操作类型

#### 中间操作`intermediate operation`

中间操作总是会惰性执行，调用中间操作只会生成一个标记了该操作的新`stream`

* `map()`将流中的每个元素应用传递进来的函数，然后再封装进流
* `flatMap()`将流中的每个元素应用传递进来的函数(该函数返回的是`stream`)，然后再将每个`stream`元素，封装进一个`stream`中
* `filter()`将流中的每个元素应用传递进来的过滤规则，`true`则通过，`false`则去掉
* `limit()`限制流通过元素的个数
* `sorted()`将流中的元素进行排序
* `distinct()`将流中的重复数据去掉

无状态操作与有状态操作

* 无状态操作(指元素的处理不受前面元素的影响)

  `unordered()`，`filter()`，`map()`，`mapToInt()`，`mapToLong()`，`mapToDouble()`，`flatMap()`，`flatMapToInt()`，`flatMapToLong()`，`flatMapToDouble()`，`peek()`

* 有状态操作(必须等到所有元素处理之后才知道最终结果)

  `distinct()`，`sorted()`，`limit()`，`skip()`

#### 最终操作`terminal operation`

最终操作会触发实际计算，计算发生时会把所有中间操作累积起来的操作以`pipeline`的方式执行，这样可以减少迭代次数，计算完成之后，`stream`就会失效

> 区分：方法的返回值为`stream`的大多都是中间操作，否则就是最终操作

* `forEach()`迭代流中的每个元素
* `count()`统计流中元素的个数
* `reduce(identity, accumulator, combiner)`，`identity`初始值，`accumulator`新元素如何累加，`combiner`多个部分如何合并
* `collect(supplier, accumulator, combiner)`，`supplier`目标容器，`accumulator`新元素如何添加到容器中，`combiner`多个部分如何合并

```java
class Student {
    int score;
    String classId;
    int sex;
    String name;

    public Student(String name, int score, String classId, int sex) {
        this.name = name;
        this.score = score;
        this.classId = classId;
        this.sex = sex;
    }

    public int getScore() {
        return score;
    }

    public String getClassId() {
        return classId;
    }

    public int getSex() {
        return sex;
    }

    @Override
    public String toString() {
        return name;
    }
}

public class Test {
    public static void main(String[] args) {
        List<Student> list = new ArrayList<>();
        list.add(new Student("f", 1, "1", 1));
        list.add(new Student("j", 2, "2", 1));
        list.add(new Student("h", 2, "1", 2));

        // 指定如何生成Map的key和value
        Map<Student, Integer> studentsAndScore = list.stream()
                .collect(Collectors.toMap(Function.identity(), Student::getScore));
        studentsAndScore.forEach((k, v) -> System.out.println(k + ":" + v));

        // 将stream中的元素依据某个二值逻辑分成互补相交的两个部分
        Map<Boolean, List<Student>> groupBySex = list.stream().collect(Collectors.partitioningBy(i -> i.getSex() > 1));
        groupBySex.forEach((k, v) -> System.out.println(k + ":" + v));

        // 按照某个属性进行分组，属性相同的元素会被对应到Map的同一个key上
        Map<String, List<Student>> groupByClassId = list.stream().collect(Collectors.groupingBy(Student::getClassId));
        groupByClassId.forEach((k, v) -> System.out.println(k + ":" + v));

        // groupingBy()允许我们分组之后再执行某种运算，这种先将元素分组的收集器叫上游收集器，之后执行其他运算的收集器叫做下游收集器
        Map<String, Long> totalByClassId = list.stream()
                .collect(Collectors.groupingBy(Student::getClassId, Collectors.counting()));
        totalByClassId.forEach((k, v) -> System.out.println(k + ":" + v));

        // 多个下游收集器
        Map<String, List<Integer>> scoreGroupByClassId = list.stream().collect(
                Collectors.groupingBy(Student::getClassId, Collectors.mapping(Student::getScore, Collectors.toList())));
        scoreGroupByClassId.forEach((k, v) -> System.out.println(k + ":" + v));

        // 字符串拼接
        Stream<String> stream = Stream.of("f", "j", "h");
        System.out.println(stream.collect(Collectors.joining(",")));
    }
}
```

非短路操作与短路操作

* 非短路操作

  `forEach()`，`forEachOrdered()`，`toArray()`，`reduce()`，`collect()`，`max()`，`min()`，`count()`

* 短路操作(指不用处理全部元素就可以返回结果)

  `anyMatch()`，`allMatch()`，`noneMatch()`，`findFirst()`，`findAny()`

### 生成流

* `stream()`创建串行流
* `parallelStream`创建并行流

### `stream`流水线的原理

#### `stream`中的基本思想

在一次迭代中尽可能多地执行用户指定的操作

#### `stream`记录操作的方式(解决用户指定的操作如何记录)

使用`Stage`的概念来描述一个完整的操作，每个`Stage`都记录了前一个`Stage`和本次的操作以及回调函数

`stream`用实例化后的`PipelineHelper`来表示`Stage`

#### `stream`叠加操作的方式(解决用户指定的操作如何叠加)

每个`Stage`都会将自己的操作和回调函数封装进`Sink`

* `Sink`接口

  * `void begin(long size)`，开始遍历元素之前调用该方法，有状态操作需要实现
  * `void end()`，所有元素遍历完成之后调用该方法，有状态操作需要实现
  * `boolean cancellationRequested()`，是否可以结束操作，短路操作返回`true`
  * `void accept(T t)`，遍历元素时调用，接受一个待处理元素，并对元素进行处理，`Stage`把自己包含的操作和回调方法封装到该方法里，前一个`Stage`的`Sink`只需要调用当前`Stage`的`Sink.accept(T t)`方法就行了

#### `stream`叠加之后的执行方式(解决如何执行)

* 上游的`Sink`如何调用下游的`Sink`

  * `PipelineHelper`中的`opWrapSink()`返回一个新的封装了当前的`Stage`操作和下游`Sink`的`Sink`
  * 从`stream`的最后一个`Stage`开始，不断调用上一个`Stage`的`opWrapSink()`直到最开始，就得到了一条包含流水线上所有操作的`Sink`

#### `stream`返回结果

|返回结果|stream api|
|:-:|:-:|
|boolean|anyMatch()，allMatch()，noneMatch()|
|Optional|findFirst()，findAny()|
|归约结果|reduce()，collect()|
|数组|toArray()|
