---
title: Stream常用场景
catalog: true
date: 2019-09-19 16:33:05
subtitle: "Stream常用场景"
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
## Stream常用场景
```
整合stream各种不同场合的使用
```
#### javaBean 
```
public class Student implements Comparable<Student>{
    private String name;
    private Integer age;

    public Student(Integer age, String name) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public int compareTo(Student student) {
        return this.age - student.getAge();
    }
}


```
#### 使用场景
```
public class SteamTest extends BaseTest{
    final static List<Integer> dataList = new ArrayList<>();
    final static List<Student> studentList = new ArrayList<>();
    final static List<String> strList = new ArrayList<>();
    private static final List<String> NAME_ORDER = Arrays.asList("jocy", "tony", "cloudy", "jack");

    {
        dataList.add(1);
        dataList.add(1);
        dataList.add(3);
        dataList.add(4);
        dataList.add(2);

        Student student1 = new Student(18, "jocy");
        Student student2 = new Student(16, "cloudy");
        Student student3 = new Student(22, "jack");
        Student student4 = new Student(53, "tony");
        studentList.add(student1);
        studentList.add(student2);
        studentList.add(student3);
        studentList.add(student4);

        strList.add("a");
        strList.add("b");
        strList.add("c");
        strList.add("d");
    }

    //计数
    @Test
    public void test1() {
        System.out.println(dataList.stream().count());
    }

    //去重
    @Test
    public void test2() {
        dataList.stream().distinct().forEach(System.out::println);
    }

    //过滤
    @Test
    public void test3() {
        dataList.stream().filter(integer -> integer > 1).forEach(System.out::println);
    }

    //过滤
    @Test
    public void test4() {
        dataList.removeIf(integer -> integer <= 1);
        dataList.forEach(System.out::println);
    }

    //合并
    @Test
    public void test5() {
        Stream.concat(dataList.stream(), strList.stream()).forEach(System.out::println);
    }

    /**
     * 数据类型转换
     * 新生成一个Stream只包含转换生成的元素
     */
    @Test
    public void test6() {
        strList.stream().map(String::toUpperCase).forEach(System.out::println);
    }

    /**
     * 数据类型转换
     * flatMap方法与map方法类似，都是将原Stream中的每一个元素通过转换函数转换，不同的是，该换转函数的对象是一个Stream
     * 不会再创建一个新的Stream，而是将原Stream的元素取代为转换的Stream
     */

    @Test
    public void test7() {
        strList.stream().flatMap(item -> Stream.of(item.toUpperCase())).forEach(System.out::println);
    }

    //过滤掉原Stream中的前N个元素，返回剩下的元素所组成的新Stream
    @Test
    public void test8() {
        dataList.stream().skip(2).forEach(System.out::println);
    }

    /**
     * 排序（升序）
     * sorted方法将对原Stream进行排序，返回一个有序列的新Stream
     * sorterd有两种变体sorted()，sorted(Comparator)
     */
    @Test
    public void test9() {
        dataList.stream().sorted().forEach(System.out::println);
    }

    /**
     * 排序（降序）
     * sorted方法将对原Stream进行排序，返回一个有序列的新Stream
     * sorterd有两种变体sorted()，sorted(Comparator)
     */
    @Test
    public void test10() {
        dataList.stream().sorted(Comparator.reverseOrder()).forEach(System.out::println);
    }


    /**
     * 排序（比较器）
     * sorted方法将对原Stream进行排序，返回一个有序列的新Stream
     * sorterd有两种变体sorted()，sorted(Comparator)
     */
    @Test
    public void test11() {
        studentList.stream().sorted(Comparator.comparing(Student::getAge)).forEach(student -> System.out.println(student.getName()));
    }

    //按字符串排序
    @Test
    public void test111() {
        List<Student> students = studentList.stream().sorted(Comparator.comparing(Student::getName, (x, y) -> {
            for (String sort : NAME_ORDER) {
                if (sort.equals(x) || sort.equals(y)) {
                    if (x.equals(y)) {
                        return 0;
                    } else if (sort.equals(x)) {
                        return -1;
                    } else {
                        return 1;
                    }
                }
            }
            return 0;
        })).collect(Collectors.toList());
    }


    /**
     * 最大值
     * Stream根据比较器Comparator，进行排序(升序或者是降序)，所谓的最大值就是从新进行排序的，
     * max就是取重新排序后的最后一个值，而min取排序后的第一个值。
     * 不管是最大值还是最小值起决定作用的是Comparator，它决定了元素比较大小的原则
     */
    @Test
    public void test12() {
        Optional<Integer> max = dataList.stream().max((o1, o2) -> o2 - o1);
        System.out.println(max.get());
    }

    /**
     * 最小值
     * Stream根据比较器Comparator，进行排序(升序或者是降序)，所谓的最大值就是从新进行排序的，
     * max就是取重新排序后的最后一个值，而min取排序后的第一个值。
     * 不管是最大值还是最小值起决定作用的是Comparator，它决定了元素比较大小的原则
     */
    @Test
    public void test13() {
        Optional<Integer> min = dataList.stream().min((o1, o2) -> o2 - o1);
        System.out.println(min.get());
    }

    /**
     * 判断Stream中的元素是否全部满足指定条件
     */
    @Test
    public void test14() {
        boolean match = dataList.stream().allMatch(integer -> integer > 2);
        System.out.println(match);
    }

    /**
     * 判断Stream中的元素是否有满足指定条件
     */
    @Test
    public void test15() {
        boolean match = dataList.stream().anyMatch(integer -> integer > 2);
        System.out.println(match);
    }

    /**
     * 此操作的行动是不确定的，其会自由的选择Stream中的任何元素
     * 在并行操作中，在同一个Stram中多次调用，可能会不同的结果。
     * 在串行调用时，Debug了几次，发现每次都是获取的第一个元素，个人感觉在串行调用时，应该默认的是获取第一个元素。
     */
    @Test
    public void test16() {
        Optional<Integer> any = dataList.stream().findAny();
        System.out.println(any.get());
    }

    /**
     * 获取第一个元素
     */
    @Test
    public void test17() {
        Optional<Integer> any = dataList.stream().findFirst();
        System.out.println(any.get());
    }

    /**
     * 截取前n位元素
     */
    @Test
    public void test18() {
        dataList.stream().limit(5).forEach(System.out::println);
    }

    /**
     * 如果所有元素都不满足条件，返回true；否则，返回false.
     */
    @Test
    public void test19() {
        boolean match = dataList.stream().noneMatch(integer -> integer > 2);
        System.out.println(match);
    }

    /**
     * collect收集器
     */
    @Test
    public void test20() {
        String collect1 = studentList.stream().map(Student::getName).collect(Collectors.joining());
        System.out.println(collect1);
        String collect2 = studentList.stream().map(Student::getName).collect(Collectors.joining(","));
        System.out.println(collect2);
        studentList.stream().map(Student::getName).collect(Collectors.toList()).forEach(System.out::println);
        studentList.stream().map(Student::getName).collect(Collectors.toSet()).forEach(System.out::println);
    }

    /**
     * 求和
     */
    @Test
    public void test21() {
        int sum = studentList.stream().mapToInt(Student::getAge).sum();
        System.out.println(sum);
    }

    /**
     * list 转map
     */
    @Test
    public void test22() {
        Map<String, List<Student>> map1 = studentList.stream().collect(Collectors.groupingBy(Student::getName));
        Map<Integer, Set<String>> map2 = studentList.stream()
            .collect(Collectors.groupingBy(Student::getAge, Collectors.mapping(Student::getName, Collectors.toSet())));
        Map<Integer, Long> map3 = studentList.stream()
            .collect(
                Collectors.groupingBy(Student::getAge, Collectors.mapping(Student::getName, Collectors.counting())));
        Map<Integer, Student> map4 = studentList.stream()
            .collect(Collectors.toMap(Student::getAge, Function.identity(), (item1, item2) -> item1));
        System.out.println(map4);
    }

}

    
```

