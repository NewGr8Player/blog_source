---
title: Java8新特性概览
categories: "Java"
tags:
  - Java
description: Java8新特性简单介绍
toc: true
date: 2018-09-14 11:04:06
---

# 接口的默认实现
Java8允许我们通过使用`default`关键字在接口中添加一个非抽象方法。
```Java
interface Formula {
    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
```
除了抽象方法`calculate`外，接口`Formula`也定义了包含默认实现的`sqrt`方法。实现类只需要实现抽象方法`calculate`。默认方法`sqrt`可以做到开箱即用。
```Java
Formula formula = new Formula() {
    @Override
    public double calculate(int a) {
        return sqrt(a * 100);
    }
};

formula.calculate(100); // 100.0
formula.sqrt(16); // 4.0
```
`formula`被作为匿名对象实现。代码看起来比较冗长，6行代码职位了实现一个简单的计算`sqrt(a * 100)`。

# Lambda表达式
首先是一个先前版本中对`List<String>`进行排序的示例
```Java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```
静态方法`Collections.sort`传入了一个`list`和`Comparator`对象用于对一直的列表进行排序。
为了不在频繁的创建匿名对象，Java8推出了更简短的语法:`lambda表达式`
```Java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

改写成Labda表达式使得代码变得更加直观简短
```Java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

对于方法体内只有一行代码的表达式，可以不使用`{}`和`return`关键字使得代码更加简短
```Java
Collections.sort(names, (a, b) -> b.compareTo(a));
```

因为Java编译器会自动推断传入参数的类型，所以参数表中的参数类型也可以省略。


# 函数式接口
lambda表达式如何匹配Java中的类型呢？每个lambda都由一个接口去指定类型。它(函数式接口)必须`仅包含一个抽象方法的声明`。每个该类型lambda表达式都会匹配这个抽象方法。

只要接口内只有一个抽象方法我们就可以把它当做lambda表达式使用。但是，为了确保接口是一定满足要求的，应当添加`@FunctionalInterface`注解。添加该注解后，当你再想接口内声明第二个抽象方法的时候，编译器会直接返回一个编译错误。
```Java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
```
**请注意，此处去掉`@FunctionalInterface`注解后该接口仍然可以作为lambda表达式使用。**

```Java
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted); // 123
```

> 一个或多个`static`方法不会影响该接口成为函数式接口
> 接口中可以包含被`default`关键字修饰的默认实现
> 支持泛型与继承关系，只要继承后的接口仍然满足成为函数式接口的条件

# 方法与构造方法的引用

之前的代码可以通过静态方法引用进一步简化：
```Java
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted); // 123
```
Java8可以使用`::`关键字传递对方法或构造方法的引用。上面的代码演示了如何引用静态方法，下面演示如何引用对象的方法。
```Java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```Java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted); // "J"
```

接下来演示调用构造方法的情况：

1. 首先定义一个拥有不同构造方法Bean
```Java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

2. 然后创建一个Person类型的工厂的接口用来创建person对象
```Java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

3. 使用示例
我们可以通过构造方法的引用去调用该接口而不用去实现其定义的抽象方法。
```Java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```
我们通过`Person::new`创建了对`Person`类构造方法的引用。Java编译器会根据`PersonFactory.create`中传入的参数重载构造方法。

# Lambda作用域
Lambda在访问外层作用域的变量时，与匿名对象的方式类似。

## 访问局部变量

```Java
final int num = 1;
Converter<Integer, String> stringConverter =
(from) -> String.valueOf(from + num);

stringConverter.convert(2); // 3
```

与匿名对象不同，变量`num`不是必须使用`final`修饰。下面代码仍然合理。

```Java
int num = 1;
Converter<Integer, String> stringConverter =
(from) -> String.valueOf(from + num);

stringConverter.convert(2); // 3
```
但是在编译过程中`num`会隐式转换为`被final修饰`的，所以如下代码不能通过编译:
```Java
int num = 1;
Converter<Integer, String> stringConverter =
(from) -> String.valueOf(from + num);
num = 3; /* 这里不同 */
```

并且在labda表达式中修改`num`的值也是**被禁止**的。

## 访问属性与静态变量

与匿名对象相同，在lambda作用域内，可以对外部的属性与静态变量进行访问与修改。
```Java
class Lambda4 {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
        outerNum = 23;
        return String.valueOf(from);
    };

    Converter<Integer, String> stringConverter2 = (from) -> {
        outerStaticNum = 72;
        return String.valueOf(from);
    };
}
}
```

## 访问接口的默认实现方法
在之前提到的`Formula`接口中声明的被`default`修饰的的`sqrt`方法可以被任意的for木旯示例包括匿名对象访问。但是对于lambda表达式则不可以。

默认方法在lambda作用域中**不能被访问**，以下代码不能正常运行。
```Java
Formula formula = (a) -> sqrt( a * 100);
```

内建函数式接口

JDK1.8中包括了许多内建的函数式接口。有些源自早先版本的JDK比如`Comparator`或者`Runnable`。这些现有的接口都是使用`@FunctionalInterface`注解拓展用来支持Lambda表达式。

Java8同样也加入了一些新的函数式接口使得开发更加快捷。比如[Google Guava](https://code.google.com/p/guava-libraries/)库。如果经常使用这个库，建议查看它的源码，以便了解其中较为常用的方法是如何通过函数式接口进行拓展的。

# 断言接口(Predicates)

`Predicates`是一个仅传入一个参数返回`boolean`类型的方法。这个接口包含了许多默认方法用于判断复杂逻辑(`且`，`或`，`非`)。

```Java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo"); // true
predicate.negate().test("foo"); // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

Functions接口(Functions)

Functions接口传入一个参数并且有返回值。默认方法可以和链式方法一起使用(`compose`,`andThen`)

```Java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123"); // "123"
```

生产者(Suppliers)

通过生产者能得到一个传入泛型的对象。与Functions接口不同，生产者不传入任何参数。

```Java
Supplier<Person> personSupplier = Person::new;
personSupplier.get(); // new Person
```

消费者(Consumers)

消费者提供对单一输入参数的处理。

```Java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

# 比较器(Comparators)

`Comparators`在早先的Java版本中比较常用。Java8为他在接口中添加了大量的默认方法。

# Optionals类(Optionals)

`Optionals`不是一个函数式接口，是一种为了避免`NullPointerException`的实用解决方案。

`Optionals`是一个简单的值容器，它可以为`null`或`非空值`。如果一个方法可能返回`null`也可能返回`非空值`就可以使用`Optional`。

```Java
Optional<String> optional = Optional.of("bam");

optional.isPresent(); // true
optional.get(); // "bam"
optional.orElse("fallback"); // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0))); // "b"
```

# 流(Streams)

`java.util.Stream`提供对一组元素进行一个或多个操作的方法。流操作不是`intermediate`(可以继续操作)就是`terminal`(不可以继续操作)的。`terminal`操作返回一个明确的类型结果，`intermediate`操作返回流本书以便于继续调用Stream方法进行处理。流处理的数据源可以来自`java.util.Collection`中的列表(list)或者集合`set`。流操作可以是连续的也可以是并行的。

```Java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```

在Java8中Collections已被拓展用于支持流操作，所以只需要调用`Collection.stream()`或`Collection.parallelStream()`就能得到流处理所需要的数据源。

## 过滤方法(Filter)

Filter传入一个`Predicate`对象用来过滤流中的所有元素。这个操作是`intermediate`(可继续操作的)。所以我们在调用filter后还可以继续调用`forEach`去遍历流中的元素。但是`forEach`是`terminal`(不可继续操作的)。它的返回值类型是`void`，所以不能再继续调用任何流处理方法。
```Java
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa2", "aaa1"
```

## 排序方法(Sorted)

Sorted是`intermediate`(可继续操作的)返回按照既定规则排序的流数据视图的方法。视图中的元素会按照`自然序`排序除非指定了`Comparator`。

```Java
stringCollection
    .stream()
    .sorted()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa1", "aaa2"
```
**`Sorted`仅创建了排序后元素的视图，并没有真正去更改集合中元素的位置，集合中元素位置不会受到该方法的影响**

```Java
System.out.println(stringCollection);
// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
```

## 映射方法(Map)

Map是`intermediate`(可继续操作的)。它通过给定方法将元素放到另一个对象中。下面的例子把流中的元素转换为大写，当然也可以使用map方法把流中对象转换为另一个类型的对象。结果流的泛型类型取决于传递给map方法的泛型类型。
```Java
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
```

## 匹配方法(Match)

`Match`是`terminal`(可继续操作的)。Match可以用来检查流中是否存在复合匹配条件的数据，并返回`true`或`false`。

```Java
boolean anyStartsWithA =
stringCollection
    .stream()
    .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA); // true

boolean allStartsWithA = 
    stringCollection 
        .stream() 
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA); // false

boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ); // true
```

## 计数器(Count)

`Count`是`terminal`(不可继续操作的)。返回流中元素的数量，返回值类型为`long`。

```Java
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();

System.out.println(startsWithB); // 3
```

## 合并(Reduce)

`Reduce`是`terminal`(不可继续操作的)。它按照给定的规则合并流内的元素，并且使用`Optional`承载结果值。

```Java
Optional<String> reduced =
    stringCollection
        .stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);
// "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
```

## 并行流(Parallel Streams)

流可以是`sequential`(顺序的)也可以是`parallel`(并行的)。并行流是在多线程的基础上完成的。

下面的例子是展示使用并行流并且并行流的使用非常简单。

1. 首先创建一个有序的并有很多元素的list

```Java
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```
> 然后我们来测试一下对这个list排序所消耗的时间。

**顺序执行排序(Sequential Sort)**
```Java
long t0 = System.nanoTime();

long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));

// sequential sort took: 899 ms
```
**并行执行排序(Parallel Sort)**
```Java
long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

// parallel sort took: 472 ms
```
> 根据演示可以很直观的看到，代码量上，两种方式几乎一样，但是并行执行的效率比顺序执行的效率要高出近50%，而在编码过程中需要做的仅仅是将`stream()`替换为`parallelStream()`。

# Map类(Map)
虽然Map现在并不支持流操作，但是它也新加入了许多好用的方法去完成一些常用需求。
```Java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));
```

上面代码中的`putIfAbsent`避免了我们在进行新增操作时候的非空校验。`forEach`则传入了一个消费者去操作map中的每个值。

下面的例子将展示如何通过函数化接口操作map
```Java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3); // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9); // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23); // true

map.computeIfAbsent(3, num -> "bam");
map.get(3); // val33
```

下面例子展示在给定key并且当前map内的value与给定值相等时才执行删除操作：
```Java
map.remove(3, "val3");
map.get(3); // val33

map.remove(3, "val33");
map.get(3); // null
```

获取给定key内的值，当不存在时，返回给定的默认值：
```Java
map.getOrDefault(42, "not found"); // not found
```

合并merging
```Java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9); // val9

map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9); // val9concat
```

> 如果合并key不冲突则直接合并，否则调用合并方法。

# 日期API(Date API)
Java8在`java.time`包下包含全新的日期和时间API。新的日期API可以与`Joda-time`库进行比较运算，但是它们并不相同。下面的示例介绍了这个新API最重要的部分。

## 时钟类(Clock)

时钟类提供对当前日期和时间的访问。时钟类依赖`时区`，可以使用它代替`System.currentTimeMillis()`来检索当前毫秒。时间线上的这样一个瞬时点也由`Instant`类表示.实例可以用来创建原有的`java.util.Date`对象。

```Java
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();

Instant instant = clock.instant();
Date legacyDate = Date.from(instant);   // legacy java.util.Date
```

## 时区类(Timezones)

时区由`ZoneId`表示。可以通过静态工厂方法访问。时区定义了非常重要的`offsets`(偏移量)使得它能够在 `instants`、`local dates` 与 `times`之间进行转化。

```Java
System.out.println(ZoneId.getAvailableZoneIds());
// prints all available timezone ids

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());
System.out.println(zone2.getRules());

// ZoneRules[currentStandardOffset=+01:00]
// ZoneRules[currentStandardOffset=-03:00]
```
## 本地时间类(LocalTime)

本地时间类不使用时区表示(指显示不包含时区)，例如：`10pm`或者`17:30:15`。下面的例子使用上面定义的时区创建了两个`LocalTime`对象。

```Java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);

System.out.println(now1.isBefore(now2));  // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween);       // -3
System.out.println(minutesBetween);     // -239
```

> `LocalTime`可以使用很多工厂去简化实例的新建过程，包括转化成字符串。

```Java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59

DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
```

## 本地日期类(LocalDate)

`LocalDate`表示一个确定的日期，比如`2014-03-11`。它与`LocalTime`类似，也是不可变的。以下例子展示如何通过年月日的加减生成新的日期。
**注意，每次操作都会返回一个新的实例。**

```Java
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
LocalDate yesterday = tomorrow.minusDays(2);

LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println(dayOfWeek);    // FRIDAY
```

从`String`(需要是符合日期格式的)转换为`LocalDate`与`LocalTime`一样简单。
```Java
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedDate(FormatStyle.MEDIUM)
        .withLocale(Locale.GERMAN);

LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
System.out.println(xmas);   // 2014-12-24
```
## 本地日期时间类(LocalDateTime)
`LocalDateTime`是日期与时间的组合。他讲上面提到的`LocalDate`与`LocalTime`组合成一个新的实例，与之相似，他也是不可变的。我们可以使用工具方法从实例中得到日期或时间相关的值。
```Java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY

Month month = sylvester.getMonth();
System.out.println(month);          // DECEMBER

long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
```

通过添加时区的操作可以使其很容易的转换为一个新的`Instant `实例。而且`Instant`很容易转换成先前版本的`java.util.Date`。
```Java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
```

格式化日期时间就像格式化日期或时间一样。我们可以根据自定义模式创建格式化程序，而无需使用预定义的格式。
```Java
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");

LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

> 与`java.text.NumberFormat`不同,`DateTimeFormatter`是`不可变的`且`线程安全的`。

# 注解(Annotations)

Annotations在Java8中是可以重复的(支持重载的)，比如下面的例子:
```Java
@interface Hints {
    Hint[] value();
}

@Repeatable(Hints.class)
@interface Hint {
    String value();
}
```

> Java8能够通过声明注解@Repeable来使用相同类型的多个注解。

**使用注解容器(老派)**
```Java
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
```

**使用Repeatable注解(新派)**
```Java
@Hint("hint1")
@Hint("hint2")
class Person {}
```

> 在使用第二种方式的时候，Java编译器会隐式的声明`@Hints`注解。

通过反射的方式获取注解：
```Java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null

Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
```

> 虽然我们从来没有在`Person`类上声明过`@Hints`注解，但它仍能通过`getAnnotation(Hints.class)`读取到。而更方便的方式是通过`getAnnotationsByType`,它能赋予所有`@Hint`注解的直接访问权限。

Java8中注解中拓展了两个新的使用目标(Target)：
```Java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
```

文章翻译自以下地址：
- [原始博客地址](https://winterbe.com/posts/2014/03/16/java-8-tutorial/)
- [github地址](https://github.com/winterbe/java8-tutorial)