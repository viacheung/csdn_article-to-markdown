---
layout:  post
title:   Spring复杂对象创建的三种方式
date:   2-01-09 17:15:22 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-spring
-java
-mysql

---


>在Spring中，对于简单类型的创建，我们可以使用set注入和构造注入。但是对于复杂类型的如何创建？

什么是复杂类型，比如连接数据库的Connection对象，以及Mybatis中的SqlSessionFactory对象。 在以前我们是通过这种方式获取Connection对象的：

```java
Connection conn = null;
        try {
   
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mysql", "root", "123456");
        } catch (ClassNotFoundException e) {
   
            e.printStackTrace();
        } catch (SQLException e) {
   
            e.printStackTrace();
        }
```

现在使用Spring如何创建这种类型的对象？Spring中提供了三种方法来创建复杂对象

## 第一种：实现FactoryBean接口

```java
public class ConnectionFactoryBean implements FactoryBean<Connection> {
   
    //用于书写创建复杂对象的代码
    @Override
    public Connection getObject() throws Exception {
   
        Class.forName(driverClassName);
        Connection conn = DriverManager.getConnection(url, username, password);
        return conn;
    }
    @Override
    public Class<?> getObjectType() {
   
        return Connection.class;
    }
    @Override
    public boolean isSingleton() {
   
        return true;
    }
    private String driverClassName;
    private String url;
    private String username;
    private String password;
	//setter and getter省略
```

在applicationContext.xml配置文件中

```java
<bean id="conn" class="com.liu.factorybean.ConnectionFactoryBean">
            <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/mysql?useSSL=false"/>
            <property name="username" value="root"/>
            <property name="password" value="123456"/>
</bean>
```


对这种使用的解读：FactoryBean接口中有三个抽象方法 ![img](https://img-blog.csdnimg.cn/9330fef7cd324436b2515e98677b1cee.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA562J5b6F6Iqx5byASQ==,size_20,color_FFFFFF,t_70,g_se,x_16) ○ 如果就想获得FactoryBean类型的对象 ctx.getBean("&amp;conn") 获得就是ConnectionFactoryBean对象 ○ isSingleton方法 &nbsp;&nbsp;返回 true 只会创建一个复杂对象 &nbsp;&nbsp;返回 false 每一次都会创建新的对象 &nbsp;&nbsp;问题：根据这个对象的特点 ，决定是返回true (SqlSessionFactory) 还是 false (Connection) ○ mysql高版本连接创建时，需要制定SSL证书，解决问题的方式 注意：类中的几个连接数据库的属性，是自己添加的，便于在配置文件中注入，实现解耦合。

## 第二种方法：实例工厂

1. 避免Spring框架的侵入 
2. 整合遗留系统 直接在这个类写创建复杂对象的方法，不用实现FactoryBean接口。

```java
public class ConnectionFactory {
   

    public Connection getConnection(){
   
        Connection conn = null;
        try {
   
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mysql", "root", "123456");
        } catch (ClassNotFoundException e) {
   
            e.printStackTrace();
        } catch (SQLException e) {
   
            e.printStackTrace();
        }
        return conn;
    }
}
```

但是要在配置文件中进行配置

```java
<!--ConnectionFactory实例 -->
 <bean id="connFactory" class="com.liu.factorybean.ConnectionFactory"></bean>
 <!--在factory-bean中应用ConnectionFactory实例id connFactory -->
 <bean id="conn"  factory-bean="connFactory" factory-method="getConnection"/>
```

## 第三种方式：静态工厂

和实例工厂类似，只不过这里把实例方法，替换为静态方法。

```java
public class StaticConnectionFactory {
   
    public static Connection getConnection(){
   
        Connection conn = null;
        try {
   
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mysql", "root", "123456");
        } catch (ClassNotFoundException e) {
   
            e.printStackTrace();
        } catch (SQLException e) {
   
            e.printStackTrace();
        }
        return conn;
    }
}
```

对应配置文件配置如下：

```java
<bean id="conn" class="com.liu.factorybean.StaticConnectionFactory" factory-method="getConnection"/>
```

## 总结：

>这些复杂对象我们在以后很少用到，因为我们在整合其他框架时，其他框架为我们提过了，创建复杂对象的方法，比如Spring整合Mybatis，Mybatis提供了创建SqlSessionFactory对象的方法。但是学习这些也是有必要的，因为这些框架底层使用的就是FactoryBean等这几种方式。

