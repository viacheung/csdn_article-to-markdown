---
layout:  post
title:   Springboot中快速使用MybatisPlus以及具体细节
date:   1-11-14 22:51:21 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Mybatis
-spring boot
-java
-intellij-idea

---


Mybatis-plus的优点，想必大家都知道。 当表中字段多的时候，我们要写很多单表的增删改查。虽然说单表的增删改查操作可以通过一些工具类或者插件来生成，但表的字段并不是一成不变的，开发的过程中总免不了要新添加新字段。使用工具就难免捉襟见肘了。 而Mybatis-Plus可以解决单表查询的痛点。 本篇博客，记录学习Mybatis-Plus的过程。



![img](https://img-blog.csdnimg.cn/129a7ebf9de1471c9189e51be3b7b266.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) 通过本篇，你可以快速学会Mybatis-Plus的使用。

## 1.新建springboot项目

## 2.加入mybatis-plus依赖

```java
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>3.4.3.4</version>
</dependency>
 <dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <scope>runtime</scope>
</dependency>
```

## 3.配置springboot文件

```java
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/springdb?useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
```

## 4.新建数据库表

新建user表


![img](https://img-blog.csdnimg.cn/img_convert/da72e4fd96ba6f7fb8eee1abf43656fa.png)

## 5.建实体类User

```java
public class User {
   
    //指定主键的方式
    //value 主键字段的名称，如果是id，可以省略
    //type:指定主键的值如何生成的
    @TableId(value = "id",type = IdType.AUTO)
    private Integer id;
    private String name;
    private String email;
    private Integer age;
    //get set省略
}
```

## 6.自定义Mapper

```java
/**
 * 自定义Mapper，Dao
 * 要实现BaseMapper
 * 指定实体类
 */
public interface UserMapper extends BaseMapper<User> {
   

}
```

## 7.扫描Mapper

在springboot启动类中，加入@Mapper注解，扫描Mapper类所在位置，生成代理Mapper类

```java
@SpringBootApplication
@MapperScan(value = "com.liu.mybatisplus.mapper")
public class MybatisPlusApplication {
   

    public static void main(String[] args) {
   
        SpringApplication.run(MybatisPlusApplication.class, args);
    }

}
```

## 8.测试

```java
@SpringBootTest
@SuppressWarnings("all")//不让usreDao报错
class MybatisPlusApplicationTests {
   

    @Autowired
    private UserMapper userDao;
    @Test
    void contextLoads() {
   
    }
    @Test
    public void insert() {
   
        User user=new User();
        user.setAge(18);
        user.setEmail("1245@qq.com");
        user.setName("张三");
        userDao.insert(user);
    }

}
```

注意：插入完之后，数据可以回显，直接可以获取自增的id值


## 库名与实体表不同时

当数据库表名与实体表不同时，需要加入@TableName注解设置


![img](https://img-blog.csdnimg.cn/img_convert/7cb9a9ec0dcae653971924c97d97a402.png)

```java
@TableName(value = "user_address")//表名
public class Address {
   
    @TableId(value = "id",type = IdType.AUTO)
    private Integer id;
    private String city;
    private String street;
    private String zipcode;
    //setter and getter
}
```

## 表字段和实体表不同

当表字段和实体表不同时


![img](https://img-blog.csdnimg.cn/img_convert/40e38440c40c96fd7e818c93e42c9f58.png)

需要在字段是加入@TableField注解

```java
@TableName(value = "user_address")
public class Address {
   
    @TableId(value = "id",type = IdType.AUTO)
    @TableField(value = "user_id")
    private Integer id;
    @TableField(value = "user_city")
    private String city;
    @TableField(value = "user_street")
    private String street;
    private String zipcode;
}
```

## 驼峰命名

上面那种方式，如果字段较多，难免显得麻烦，采用驼峰命名的话就不用写，@TableFiled


![img](https://img-blog.csdnimg.cn/img_convert/f416d99af1394c0c9b6eb41a666583c0.png)

对应的java实体类可以这样写

```java
@TableName("customer")
public class Customer {
   
    @TableId(value = "id",type = IdType.AUTO)
    private Integer id;
    private String custName;
    private int custAge;
    private String custEmail;
}
```


## 使用配置文件

在XxxMapper文件中自定义sql方法


![img](https://img-blog.csdnimg.cn/img_convert/e5bb262395e0438e8a7f9f52af77f961.png)

在resource文件夹新建mapper文件


![img](https://img-blog.csdnimg.cn/img_convert/6d0dde9e22db68a0b743d7f472f5b746.png)

springboot配置文件中加入配置文件的位置：

```java
mybatis-plus:
  mapper-locations: classpath*:mapper/*Mapper.xml
```

## 使用注解

直接在XxxMapper文件中写入sql语句


![img](https://img-blog.csdnimg.cn/img_convert/1fc1ab66c8870a019025319bbb9cb4a1.png)


在mybatis-plus当中，有一个Wrapper类，可以封装查询和更新条件


![img](https://img-blog.csdnimg.cn/img_convert/ac8013ff4236eeeb9f520cbde3f91313.png)

QueryQrapper中常用方法

## allEq()

```java
@Test
public void testAlleq2(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    Map<String,Object> parm=new HashMap<>();
    parm.put("name","张三");
    parm.put("age",null);
    //SELECT id,name,age,email,status FROM student WHERE (name = ? AND age IS NULL)
    qw.allEq(parm,true);
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## eq()等于

```java
@Test
public void testeq(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    qw.eq("name","张三");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## ne()不等于

```java
@Test
public void testne(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (name <> ?)
    qw.ne("name","张三");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## gt()大于

ge()大于等于

```java
@Test
public void testgt(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (age > ?)
    qw.gt("age",22);
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## lt()小于

le()小于等于

```java
@Test
public void testlt(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (age < ?)
    qw.lt("age",22);
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## between

```java
@Test
public void testBetween(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (age BETWEEN ? AND ?)
    qw.between("age",22,36);
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## notBetween

```java
@Test
public void testNotBetween(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (age NOT BETWEEN ? AND ?)
    qw.notBetween("age",22,35);
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## like notLike模糊查询

```java
@Test
public void testLike(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (name LIKE ?) %张%
    qw.like("name","张");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
@Test
public void testNotLike(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (name NOT LIKE ?) %张%
    qw.notLike("name","张");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## 部分匹配likeLeft，likeRight

likeRight()

```java
@Test
public void testLikeRight(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (name LIKE ?) 张%(String)
    qw.likeRight("name","张");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## isNull isNotNull

判断字段是不是空值

```java
@Test
public void testIsNull(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (email IS NULL)
    qw.isNull("email");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## in notIN

```java
@Test
public void testIn(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (name IN (?,?))
    //也可以使用集合
    qw.in("name","张三","李四");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## inSql notInsql

inSql()可以用来做子查询，类似in()

```java
@Test
public void testInSql(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (age IN (select age from student where id =1))
    //insql相当于in里面的子查询
    qw.inSql("age","select age from student where id =1");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
}
```

## Groupby

```java
@Test
public void testGroupby(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT name,count(*) personNumbers FROM student GROUP BY name
    qw.select("name,count(*) personNumbers");//select name,count(*) personNumbers
    QueryWrapper<Student> name = qw.groupBy("name");
    List<Student> studentList = studentMapper.selectList(qw);
    System.out.println(studentList);
    System.out.println(name);
}
```

## OrderBy

```java
@Test
public void testOrder(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student ORDER BY name ASC,age DESC
    qw.orderByAsc("name");
    qw.orderByDesc("age");
    List<Student> students = studentMapper.selectList(qw);
    System.out.println(students);
}
```

## Or

```java
@Test
public void testOrAnd(){
   
    QueryWrapper<Student> qw=new QueryWrapper<>();
    //SELECT id,name,age,email,status FROM student WHERE (name = ? OR age = ?)
    qw.eq("name","张三")
        .or()
        .eq("age",22);
    List<Student> students = studentMapper.selectList(qw);
    System.out.println(students);
}
```

## Mybatis-Plus分页

根据Mybatis-Plus官网


![img](https://img-blog.csdnimg.cn/img_convert/ac809f61482a6451a8ffa56352ad771f.png)

首先要定义配置类，其实就是拦截器

```java
@Configuration
public class MybatisPlusConfig {
   
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
   
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

然后就可以直接使用，其中IPage是Mybatis-Plus分页核心类

ipage对象中可以设置分页的信息，每页大小，当前页。

分页时，首先会两条sql语句，首先会查询一共有多少条数据，然后第二条sql才是要查询的内容

```java
@Test
public void testPage(){
   
    //SELECT COUNT(*) AS total FROM student WHERE (age > ?)
    //SELECT id,name,age,email,status FROM student WHERE (age > ?) LIMIT ?
    QueryWrapper<Student> qw=new QueryWrapper<>();
    qw.gt("age",22);
    IPage<Student> page =new Page<>();
    page.setCurrent(1);
    page.setSize(3);
    IPage<Student> result = studentMapper.selectPage(page, qw);
    //records包含了分页的查询结果
    List<Student> records = result.getRecords();
    System.out.println(result.getSize());
    System.out.println(result.getCurrent());
    System.out.println(result.getTotal());
}
```

