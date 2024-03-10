https://blog.csdn.net/mu_wind/article/details/109516995

![image.png](https://note.youdao.com/yws/res/46474/WEBRESOURCE614b09f18afa85ac1f5f42d1dc501048)

#### 1 Stream概述

Stream可以由数组或集合创建，对流的操作分为两种：
* 中间操作，每次返回一个新的流，可以有多个。
* 终端操作，每个流只能进行一次终端操作，终端操作结束后流无法再次使用。终端操作会产生一个新的集合或值。


Stream有几个特性：
* stream不存储数据，而是按照特定的规则对数据进行计算，一般会输出结果。
* stream不会改变数据源，通常情况下会产生一个新的集合或一个值。
* stream具有延迟执行特性，只有调用终端操作时，中间操作才会执行。

#### 2 Stream的创建

1、通过 java.util.Collection.stream() 方法用集合创建流
```java
List<String> list = Arrays.asList("a", "b", "c");
// 创建一个顺序流
Stream<String> stream = list.stream();
// 创建一个并行流
Stream<String> parallelStream = list.parallelStream();

```

2、使用java.util.Arrays.stream(T[] array)方法用数组创建流
```java
int[] array={1,3,5,6,8};
IntStream stream = Arrays.stream(array);
```

3、使用Stream的静态方法：of()、iterate()、generate()
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);

Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 3).limit(4);
stream2.forEach(System.out::println);

Stream<Double> stream3 = Stream.generate(Math::random).limit(3);
stream3.forEach(System.out::println);
```

Stream和parallelStream的**简单区分**： stream是顺序流，由主线程按顺序对流执行操作，而parallelStream是并行流，内部以多线程并行执行的方式对流进行操作，但前提是流中的数据处理没有顺序要求

#### 3 Stream的使用

#### 3.1 遍历/匹配(foreach/find/match)

```java
List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);

// 遍历输出符合条件的元素
list.stream().filter(x -> x > 6).forEach(System.out::println);
// 匹配第一个
Optional<Integer> findFirst = list.stream().filter(x -> x > 6).findFirst();
// 匹配任意（适用于并行流）
Optional<Integer> findAny = list.parallelStream().filter(x -> x > 6).findAny();
// 是否包含符合特定条件的元素
boolean anyMatch = list.stream().anyMatch(x -> x > 6);
System.out.println("匹配第一个值：" + findFirst.get());
```

#### 3.2 筛选(filter)

```java
List<Integer> list = Arrays.asList(6, 7, 3, 8, 1, 2, 9);
Stream<Integer> stream = list.stream();
stream.filter(x -> x > 7).forEach(System.out::println);


List<Person> personList = new ArrayList<Person>();
personList.add(new Person("Tom", 8900, "male", "New York"));
personList.add(new Person("Jack", 7000, "male", "Washington"));
personList.add(new Person("Lily", 7800, "female", "Washington"));
personList.add(new Person("Anni", 8200, "female", "New York"));
personList.add(new Person("Owen", 9500, "male", "New York"));
personList.add(new Person("Alisa", 7900, "female", "New York"));
List<String> fiterList = personList.stream().filter(x -> x.getSalary() > 8000).map(Person::getName)
        .collect(Collectors.toList());
System.out.print("薪资高于8000美元的员工：" + fiterList);
```

#### 3.3 聚合(max/min/count)

```java
//获取string集合最长的元素
List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");
Optional<String> max = list.stream().max(Comparator.comparing(String::length));
System.out.println("最长的字符串：" + max.get());
```

```java
//获取Integer集合中最大值
List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);
// 自然排序
Optional<Integer> max = list.stream().max(Integer::compareTo);
// 自定义排序（从大到小排序）
Optional<Integer> max2 = list.stream().max((o1, o2) -> o2 - o1);
System.out.println("自然排序的最大值：" + max.get());
System.out.println("自定义排序的最大值：" + max2.get());
//自然排序的最大值：11
//自定义排序的最大值：4
```

```java
//获取薪资最高的人
Optional<Person> max = personList.stream().max(Comparator.comparingInt(Person::getSalary));
System.out.println("员工薪资最大值：" + max.get().getSalary());
```

```java
List<Integer> list = Arrays.asList(7, 6, 4, 8, 2, 11, 9);
long count = list.stream().filter(x -> x > 6).count();
System.out.println("list中大于6的元素个数：" + count);
```

#### 3.4 映射(map/flatMap)

映射，可以将一个流的元素按照一定的映射规则映射到另一个流中

* map：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
* flatMap：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

```java
String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
List<String> strList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());

List<Integer> intList = Arrays.asList(1, 3, 5, 7, 9, 11);
List<Integer> intListNew = intList.stream().map(x -> x + 3).collect(Collectors.toList());

System.out.println("每个元素大写：" + strList);
System.out.println("每个元素+3：" + intListNew);
```

```java
// 不改变原来员工集合的方式
List<Person> personListNew = personList.stream().map(person -> {
    Person personNew = new Person(person.getName(), 0, 0, null, null);
    personNew.setSalary(person.getSalary() + 10000);
    return personNew;
}).collect(Collectors.toList());
System.out.println("一次改动前：" + personList.get(0).getName() + "-->" + personList.get(0).getSalary());
System.out.println("一次改动后：" + personListNew.get(0).getName() + "-->" + personListNew.get(0).getSalary());

// 改变原来员工集合的方式
List<Person> personListNew2 = personList.stream().map(person -> {
    person.setSalary(person.getSalary() + 10000);
    return person;
}).collect(Collectors.toList());
System.out.println("二次改动前：" + personList.get(0).getName() + "-->" + personListNew.get(0).getSalary());
System.out.println("二次改动后：" + personListNew2.get(0).getName() + "-->" + personListNew.get(0).getSalary());

```

```java
//将两个字符组合并成一个新的字符数组
List<String> list = Arrays.asList("m,k,l,a", "1,3,5,7");
List<String> listNew = list.stream().flatMap(s -> {
    // 将每个元素转换成一个stream
    String[] split = s.split(",");
    Stream<String> s2 = Arrays.stream(split);
    return s2;
}).collect(Collectors.toList());

System.out.println("处理前的集合：" + list);
System.out.println("处理后的集合：" + listNew);
```

#### 3.5 归约(reduce)

归约，也称缩减，顾名思义，是把一个流缩减成一个值，能实现对集合求和、求乘积和求最值操作。

```java
//求Integer集合的元素之和、乘积和最大值
List<Integer> list = Arrays.asList(1, 3, 2, 8, 11, 4);
// 求和方式1
Optional<Integer> sum = list.stream().reduce((x, y) -> x + y);
// 求和方式2
Optional<Integer> sum2 = list.stream().reduce(Integer::sum);
// 求和方式3
Integer sum3 = list.stream().reduce(0, Integer::sum);

// 求乘积
Optional<Integer> product = list.stream().reduce((x, y) -> x * y);

// 求最大值方式1
Optional<Integer> max = list.stream().reduce((x, y) -> x > y ? x : y);
// 求最大值写法2
Integer max2 = list.stream().reduce(1, Integer::max);

System.out.println("list求和：" + sum.get() + "," + sum2.get() + "," + sum3);
System.out.println("list求积：" + product.get());
System.out.println("list求最大值：" + max.get() + "," + max2);
```

```java
// 求工资之和方式1：
Optional<Integer> sumSalary = personList.stream().map(Person::getSalary).reduce(Integer::sum);
// 求工资之和方式2：
Integer sumSalary2 = personList.stream().reduce(0, (sum, p) -> sum += p.getSalary(),
        (sum1, sum2) -> sum1 + sum2);
// 求工资之和方式3：
Integer sumSalary3 = personList.stream().reduce(0, (sum, p) -> sum += p.getSalary(), Integer::sum);

// 求最高工资方式1：
Integer maxSalary = personList.stream().reduce(0, (max, p) -> max > p.getSalary() ? max : p.getSalary(),
        Integer::max);
// 求最高工资方式2：
Integer maxSalary2 = personList.stream().reduce(0, (max, p) -> max > p.getSalary() ? max : p.getSalary(),
        (max1, max2) -> max1 > max2 ? max1 : max2);
// 求最高工资方式3：
Integer maxSalary3 = personList.stream().map(Person::getSalary).reduce(Integer::max).get();

System.out.println("工资之和：" + sumSalary.get() + "," + sumSalary2 + "," + sumSalary3);
System.out.println("最高工资：" + maxSalary + "," + maxSalary2 + "," + maxSalary3);
```

#### 3.6 收集(collect)

###### 3.6.1 归集(toList/toSet/toMap)

```java
List<Integer> list = Arrays.asList(1, 6, 3, 4, 6, 7, 9, 6, 20);
List<Integer> listNew = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
Set<Integer> set = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toSet());

Map<?, Person> map = personList.stream().filter(p -> p.getSalary() > 8000)
        .collect(Collectors.toMap(Person::getName, p -> p));
System.out.println("toList:" + listNew);
System.out.println("toSet:" + set);
System.out.println("toMap:" + map);
```

##### 3.6.2 统计(count/averaging)

* 计数 count
* 平均值 averagingInt、averagingLong、averagingDouble
* 最值 maxBy、minBy
* 求和 summingInt、summingLong、summingDouble
```java
// 求总数
Long count = personList.stream().collect(Collectors.counting());
// 求平均工资
Double average = personList.stream().collect(Collectors.averagingDouble(Person::getSalary));
// 求最高工资
Optional<Integer> max = personList.stream().map(Person::getSalary).collect(Collectors.maxBy(Integer::compare));
// 求工资之和
Integer sum = personList.stream().collect(Collectors.summingInt(Person::getSalary));
// 一次性统计所有信息
DoubleSummaryStatistics collect = personList.stream().collect(Collectors.summarizingDouble(Person::getSalary));

System.out.println("员工总数：" + count);
System.out.println("员工平均工资：" + average);
System.out.println("员工工资总和：" + sum);
System.out.println("员工工资所有统计：" + collect);
```

##### 3.6.3 分组(partitioningBy/groupingBy)

```java
// 将员工按薪资是否高于8000分组
Map<Boolean, List<Person>> part = personList.stream().collect(Collectors.partitioningBy(x -> x.getSalary() > 8000));
// 将员工按性别分组
Map<String, List<Person>> group = personList.stream().collect(Collectors.groupingBy(Person::getSex));
// 将员工先按性别分组，再按地区分组
Map<String, Map<String, List<Person>>> group2 = personList.stream().collect(Collectors.groupingBy(Person::getSex, Collectors.groupingBy(Person::getArea)));
System.out.println("员工按薪资是否大于8000分组情况：" + part);
System.out.println("员工按性别分组情况：" + group);
System.out.println("员工按性别、地区：" + group2);
```

##### 3.6.4 接合(joining)

```java
String names = personList.stream().map(p -> p.getName()).collect(Collectors.joining(","));
System.out.println("所有员工的姓名：" + names);
List<String> list = Arrays.asList("A", "B", "C");
String string = list.stream().collect(Collectors.joining("-"));
System.out.println("拼接后的字符串：" + string);
```

##### 3.6.5 归约(reducing)

Collectors类提供的reducing方法，相比于stream本身的reduce方法，增加了对自定义归约的支持。

```java
// 每个员工减去起征点后的薪资之和（这个例子并不严谨，但一时没想到好的例子）
Integer sum = personList.stream().collect(Collectors.reducing(0, Person::getSalary, (i, j) -> (i + j - 5000)));
System.out.println("员工扣税薪资总和：" + sum);

// stream的reduce
Optional<Integer> sum2 = personList.stream().map(Person::getSalary).reduce(Integer::sum);
System.out.println("员工薪资总和：" + sum2.get());
```

#### 3.7 排序(sorted)

sorted，中间操作。有两种排序：

* sorted()：自然排序，流中元素需实现Comparable接口
* sorted(Comparator com)：Comparator排序器自定义排序

```java
// 按工资升序排序（自然排序）
List<String> newList = personList.stream().sorted(Comparator.comparing(Person::getSalary)).map(Person::getName)
		.collect(Collectors.toList());
// 按工资倒序排序
List<String> newList2 = personList.stream().sorted(Comparator.comparing(Person::getSalary).reversed())
		.map(Person::getName).collect(Collectors.toList());
// 先按工资再按年龄升序排序
List<String> newList3 = personList.stream()
		.sorted(Comparator.comparing(Person::getSalary).thenComparing(Person::getAge)).map(Person::getName)
		.collect(Collectors.toList());
// 先按工资再按年龄自定义排序（降序）
List<String> newList4 = personList.stream().sorted((p1, p2) -> {
	if (p1.getSalary() == p2.getSalary()) {
		return p2.getAge() - p1.getAge();
	} else {
		return p2.getSalary() - p1.getSalary();
	}
}).map(Person::getName).collect(Collectors.toList());

System.out.println("按工资升序排序：" + newList);
System.out.println("按工资降序排序：" + newList2);
System.out.println("先按工资再按年龄升序排序：" + newList3);
System.out.println("先按工资再按年龄自定义降序排序：" + newList4);
```

#### 3.8 提取/组合

```java
String[] arr1 = { "a", "b", "c", "d" };
String[] arr2 = { "d", "e", "f", "g" };

Stream<String> stream1 = Stream.of(arr1);
Stream<String> stream2 = Stream.of(arr2);
// concat:合并两个流 distinct：去重
List<String> newList = Stream.concat(stream1, stream2).distinct().collect(Collectors.toList());
// limit：限制从流中获得前n个数据
List<Integer> collect = Stream.iterate(1, x -> x + 2).limit(10).collect(Collectors.toList());
// skip：跳过前n个数据
List<Integer> collect2 = Stream.iterate(1, x -> x + 2).skip(1).limit(5).collect(Collectors.toList());

System.out.println("流合并：" + newList);
System.out.println("limit：" + collect);
System.out.println("skip：" + collect2);
```
