---
layout:  post
title:   基于java8函数式接口Function实现MyBatisUtils
date:   1-10-29 16:52:57 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Mybatis
-java
-开发语言
-后端

---


```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
import java.io.Reader;
import java.util.function.Function;

public class MybatisUtils {
   
    //利用static属于类，不属于对象，且全局唯一
    private static SqlSessionFactory sqlSessionFactory=null;
    //利用静态块在初始化类时实例化sqlSessionFactory
    static {
   
        Reader reader=null;
        try{
   
            reader= Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory =new SqlSessionFactoryBuilder().build(reader);
        }catch (IOException e){
   
            throw new ExceptionInInitializerError(e);
        }
    }

    /**
     * 执行select查询sql
     * @param func 要执行查询语句的代码块
     * @return 查询结果
     */
    public static Object executeQuery(Function<SqlSession,Object> func){
   
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try {
   
            Object obj = func.apply(sqlSession);
            return obj;
        }finally {
   
            sqlSession.close();
        }
    }

    /**
     * 执行insert/update/delete写操作
     * @param func 要执行的操作代码块
     * @return 写操作结果
     */
    public static Object executeUpdate(Function<SqlSession,Object> func){
   
        SqlSession sqlSession = sqlSessionFactory.openSession(false);
        try {
   
            Object obj = func.apply(sqlSession);
            sqlSession.commit();
            return obj;
        }catch (RuntimeException e){
   
            sqlSession.rollback();
            throw e;
        }finally {
   
            sqlSession.close();
        }
    }

}
```

