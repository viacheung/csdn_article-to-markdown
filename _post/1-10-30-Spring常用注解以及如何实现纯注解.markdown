---
layout:  post
title:   Spring常用注解以及如何实现纯注解
date:   1-10-30 10:04:49 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Spring
-spring
-java
-java-ee

---


```java
@Around(value="execution(* login(..))")
环绕通知,value里面是切入点表达式
```

```java
@Component("id") 
替换原有的<bean>标签
 衍生注解 @Component
@Service 用于Service层
@Repository 用于Dao层
@Controller 用户控制层
更加准确的表达一个类型的作用
```

```java
@Scope 控制对象创建次数
作用：控制简单对象创建次数
注意：不添加@Scope Spring提供默认值 singleton
<bean id="" class="" scope="singleton|prototype"/>
```

```java
@Lazy注解
作用：延迟创建单实例对象
注意：一旦使用了@Lazy注解后，Spring会在使用这个对象时候，进行这个对象的创建
<bean id="" class="" lazy="false"/>
```

```java
生命周期方法相关注解 
1. 初始化相关方法 @PostConstruct
   InitializingBean
   <bean init-method=""/>
2. 销毁方法 @PreDestroy
   DisposableBean
   <bean destory-method=""/>
注意：1. 上述的2个注解并不是Spring提供的，JSR(JavaEE规范)520
     2. 再一次的验证，通过注解实现了接口的契约性
```

为jdk类型赋值

```java
1.设置xxx.properties
2.Spring工场读取这个配置文件
	<context:property-placeholder location="" />
	@PropertySource替换标签配置
3. @Value("${key}")
	int id;
```

```java
<tx:annotation-driven />
<context:component-scan base-package="com.liu"/> ==>@ComponentScan 进行相关注解的扫描，使其生效
```

```java
@Configuration //用于替换XML配置文件
public class AppConfig1{
   
  
}
@Configuration
@ImportResource("applicationContext.xml")//导入配置文件
@Import("AppCongi1.class") //导入其他配置类
public class AppConfig2{
   
  
}
```

```java
@Bean注解
@Bean注解在配置bean中进行使用，等同于XML配置文件中的<bean标签
    
@Bean("id")
@Scope
public Connection conn() {
   
  Connection conn = null;
  try {
   
    ConnectionFactoryBean factoryBean = new ConnectionFactoryBean();
    conn = factoryBean.getObject();
  } catch (Exception e) {
   
    e.printStackTrace();
  }
  return conn;
}
```

```java
jdk类型赋值
@Configuration
@PropertySource("classpath:/init.properties")
public class AppConfig1 {
   
    @Value("${id}")
    private Integer id;
    @Value("${name}")
    private String name;
 
    @Bean
    public Customer customer() {
   
        Customer customer = new Customer();
        customer.setId(id);
        customer.setName(name);

        return customer;
    }
}
```

```java
配置优先级
如果不满意，只要id相同，可以进行覆盖，只添加，不修改
@Component及其衍生注解 < @Bean < 配置文件bean标签
```

```java
<aop:aspectj-autoproxy proxy-target-class=true|false /> //为spring容器中那些配置@aspectJ切面的bean创建代理，织入切面
使用纯注解编程后，可以用以下注解替换
@EnableAspectjAutoProxy(proxyTargetClass="true|flase") ---> 写在配置文件当中Bean

SpringBoot AOP的开发方式
     @EnableAspectjAutoProxy 已经设置好了
Spring AOP 代理默认实现 JDK  SpringBOOT AOP 代理默认实现 Cglib
```

纯注解编程

```java
1. 连接池
  <!--连接池-->
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/suns?useSSL=false"></property>
    <property name="username" value="root"></property>
    <property name="password" value="123456"></property>
  </bean>
   
   @Bean
   public DataSource dataSource(){
      DruidDataSource dataSource = new DruidDataSource();
      dataSource.setDriverClassName("");
      dataSource.setUrl();
      ...
      return dataSource;
   }

2. SqlSessionFactoryBean
    <!--创建SqlSessionFactory SqlSessionFactoryBean-->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="dataSource"></property>
      <property name="typeAliasesPackage" value="com.baizhiedu.entity"></property>
      <property name="mapperLocations">
        <list>
          <value>classpath:com.baizhiedu.mapper/*Mapper.xml</value>
        </list>
      </property>
    </bean>

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
         SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
         sqlSessionFactoryBean.setDataSource(dataSource);
         sqlSessionFactoryBean.setTypeAliasesPackage("");
         ...
         return sqlSessionFactoryBean;
    }

3. MapperScannerConfigure 
   <!--创建DAO对象 MapperScannerConfigure-->
  <bean id="scanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"></property>
    <property name="basePackage" value="com.baizhiedu.dao"></property>
  </bean>
  对于MapperConfigurespring提供了专属得注解
  @MapperScan(basePackages={"com.baizhiedu.dao"}) ---> 配置bean完成
	会自动扫描SqlsessionFactoryBean
```

