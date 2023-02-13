---
layout:  post
title:   Lambda和Stream练习
date:   1-08-27 09:08:22 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-java

---


```java
-  一个 Lambda 表达式可以有零个或多个参数 参数的类型既可以明确声明，也可以根据上下文来推断。
-	所有参数需包含在圆括号内，参数之间用逗号相隔。 空圆括号代表参数集为空。  	当只有一个参数，且其类型可推导时，圆括号（）可省略。
-	Lambda 表达式的主体可包含零条或多条语句 如果 Lambda表达式的主体只有一条语句，花括号{}可省略。
 -	匿名函数的返回类型与该主体表达式一致 如果    Lambda表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空
```

```java
import org.junit.Test;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Locale;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
public class LambdaText {
   
    @Test
    public void Test01(){
   
        op(100L,200L,(x,y)->x+y);
        happy(100.0,(d)-> System.out.println("消费:"+d) );
    }
    @Test
    public void Test02(){
   
        String hander = strHander("hgdjlgf", (str) -> str.toUpperCase());
        System.out.println(hander);
    }
    public void op(Long o1,Long o2,Fun<Long,Long> my){
   
        System.out.println(my.getValue(o1, o2));
    }
    public void happy(Double money, Consumer<Double> con){
   
        con.accept(money);
    }
    public String strHander(String str, Function<String,String> fun){
   
        return fun.apply(str);
    }
    @Test
    public  void test03(){
   
        List<Character> list = predict("abcdefg", (x) -> x > 99);
        list.forEach(System.out::println);
    }
    public List<Character> predict(String str, Predicate<Character> predicate){
   
        List<Character> list=new ArrayList<>();
        for (Character c:str.toCharArray()){
   
            if (predicate.test(c)){
   
                list.add(c);
            }
        }
        return list;
    }
    @Test
    public  void test04(){
   
        Comparator<Integer> com=(x,y)->Integer.compare(x,y);
        int compare = com.compare(3, 4);
        System.out.println(compare);
        Comparator<Integer> com1=Integer::compare;

    }
    @Test
    public void Test05(){
   
        Function<Integer,Person> fun=(x)->new Person(x);
        System.out.println(fun.apply(18));
        BiFunction<Integer,String,Person> bf=(x,y)->new Person(x,y);

        BiFunction<Integer,String,Person> bf1=Person::new;
        System.out.println(bf.apply(18, "liu"));
    }

    @Test
    public void Test06(){
   
        Function<Integer,Integer[]> fun=(x)->new Integer[x];

        Function<Integer,Integer[]> fun1=Integer[]::new;

        System.out.println(fun1.apply(8).length);
    }
}
```

```java
import org.junit.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

//获取Stream流的四种方式
public class StreamApiTest {
   
    List<Employee> emps = Arrays.asList(
            new Employee(101, "张三", 18, 9999.99),
            new Employee(102, "李四", 59, 6666.66),
            new Employee(103, "王五", 28, 3333.33),
            new Employee(104, "赵六", 8, 7777.77),
            new Employee(105, "田七", 38, 5555.55),
            new Employee(105, "田七", 38, 5555.55)
    );
    @Test
    public void test01(){
   
        //1.通过Collection获取Stream
        List<String> list=new ArrayList<>();
        Stream<String> stream = list.stream();

        //2.通过Arrays.stream()获取数组流
        Stream<Integer> stream1 = Arrays.stream(new Integer[5]);

        //3.通过Stream.of获取流
        Stream<String> stream2 = Stream.of("abc", "def");

        //创建无限流
        Stream<Integer> stream3 = Stream.iterate(1, (x) -> x + 1);
        stream3.limit(10)
                .forEach(System.out::println);
    }

    @Test
    public void test02(){
   
        emps.stream()
                .distinct()
                .filter((x) -> x.getSalary() < 5556.0)
                .forEach(System.out::println);
    }
    @Test
    public void test03(){
   
        emps.stream()
                .map(Employee::getName)
                .distinct()
                .forEach(System.out::println);
    }
    @Test
    public void test4() {
   
        List<String> list = Arrays.asList("aaa", "bbb", "ccc", "ddd", "eee");
        //map—接收Lambda，将元素转换成其他形式或提取信息。接收一个函数作为参数，
        // 该函数会被应用到每个元素上，并将其映射成一个新的元素。
        list.stream().map((str)->str.toUpperCase()).forEach(System.out::println);
        System.out.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~");
        emps.stream().map(Employee::getName).forEach(System.out::println);

        System.out.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~");
        //扁平化 flatMap将多个流整合成一个流 类似于list中的add 和 addAll
        Stream<Stream<Character>> streamStream = 	list.stream().map(this::filterCharacter);
        Stream<Character> characterStream = list.stream().flatMap(this::filterCharacter);
        characterStream.forEach(System.out::println);
    }

    public  Stream<Character> filterCharacter(String str){
   
        List<Character> list = new ArrayList<>();
        for (Character c : str.toCharArray()) {
   
            list.add(c);
        }
        return list.stream();
    }

    @Test//排序
    public void test5() {
   
        emps.stream()
            .sorted((o1,o2)->Integer.compare(o1.getAge(),o2.getAge()))
            .forEach(System.out::println);
    }

}
```

