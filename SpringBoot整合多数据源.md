# SpringBoot整合多数据源

思考：在项目中使用多数据源？

![1570157151099](.\assets\1570157151099.png)

==原理使用根据包名，加载不同的数据源。==



### （1）、application.properties新增来个数据连接属性

```properties
###datasource1
spring.datasource.test1.driver-class-name = com.mysql.jdbc.Driver
spring.datasource.test1.jdbc-url = jdbc:mysql://localhost:3306/itmy_test01?useUnicode=true&characterEncoding=utf-8
spring.datasource.test1.username = root
spring.datasource.test1.password = root
###datasource2
spring.datasource.test2.driver-class-name = com.mysql.jdbc.Driver
spring.datasource.test2.jdbc-url = jdbc:mysql://localhost:3306/itmy_test02?useUnicode=true&characterEncoding=utf-8
spring.datasource.test2.username = root
spring.datasource.test2.password = root


```



### (2)、配置文件中新增两个数据源


DataSource1Config.java

```java
package com.nieshenkuan.datasource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 20:46
 */
// DataSource01
@Configuration // 注册到springboot容器中
@MapperScan(basePackages = "com.nieshenkuan.test01", sqlSessionFactoryRef = "test1SqlSessionFactory")
public class DataSource1Config {

    /**
     *@Description: 功能描述:(配置test01数据库)
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/10/1 20:47
     */
    @Bean(name = "test1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.test1")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }


    /**
     *@Description: 功能描述:(test1 sql会话工厂)
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/10/1 20:48
     */
    @Bean(name = "test1SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        // bean.setMapperLocations(
        // new
        // PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test1/*.xml"));
        return bean.getObject();
    }


    /**
     *@Description: 功能描述:(test1 事物管理)
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/10/1 20:48
     */
    @Bean(name = "test1TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test1SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}


```



DataSource2Config.java

```java
package com.nieshenkuan.datasource;
import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 20:49
 */
// DataSource02
@Configuration // 注册到springboot容器中
@MapperScan(basePackages = "com.nieshenkuan.test02", sqlSessionFactoryRef = "test2SqlSessionFactory")
public class DataSource2Config {

    /**
     *@Description: 功能描述:(配置test01数据库)
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/10/1 20:51
     */
    @Bean(name = "test2DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.test2")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    /**
     *@Description: 功能描述:(test2 sql会话工厂)
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/10/1 20:51
     */
    @Bean(name = "test2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test2DataSource") DataSource dataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        // bean.setMapperLocations(
        // new
        // PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test2/*.xml"));
        return bean.getObject();
    }


    /**
     *@Description: 功能描述:(test2 事物管理)
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/10/1 20:51
     */
    @Bean(name = "test2TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test2SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("test2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}


```



### (3)、创建分包Mapper



#### 3.1、test01包下

UserMapperTest01.java

```java
package com.nieshenkuan.test01.mapper;

import com.nieshenkuan.entity.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 20:42
 */
public interface UserMapperTest01 {
    // 查询语句
    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    User findByName(@Param("name") String name);

    // 添加
    @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);
}

```



UserServiceTest01.java

```java
package com.nieshenkuan.test01.service;

import com.nieshenkuan.test01.mapper.UserMapperTest01;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 20:43
 */
@Service
@Slf4j
public class UserServiceTest01 {
    @Autowired
    private UserMapperTest01 userMapperTest01;

    @Transactional(transactionManager = "test1TransactionManager")
    public int insertUser(String name, Integer age) {
        int insertUserResult = userMapperTest01.insert(name, age);
        log.info("######insertUserResult:{}##########", insertUserResult);
        int i = 1 / age;
        // 怎么样验证事务开启成功!~
        return insertUserResult;
    }

}

```



#### 3.2、test02下

UserMapperTest02.java

```java
package com.nieshenkuan.test02.mapper;

import com.nieshenkuan.entity.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 20:44
 */
public interface UserMapperTest02 {
    // 查询语句
    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    User findByName(@Param("name") String name);

    // 添加
    @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);
}

```

UserServiceTest02.java

```java
package com.nieshenkuan.test02.service;

import com.nieshenkuan.test02.mapper.UserMapperTest02;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 20:45
 */
@Service
@Slf4j
public class UserServiceTest02 {
    @Autowired
    private UserMapperTest02 userMapperTest02;

    @Transactional(transactionManager = "test2TransactionManager")
    public int insertUser(String name, Integer age) {
        int insertUserResult = userMapperTest02.insert(name, age);
        log.info("######insertUserResult:{}##########", insertUserResult);
        // 怎么样验证事务开启成功!~
        int i = 1 / age;
        return insertUserResult;
    }

}

```



### (4)、多数据源事务注意事项

在多数据源的情况下，使用@Transactional注解时，应该指定事务管理者

@Transactional(transactionManager = "test2TransactionManager")



### （5）、启动项目

```java
package com.nieshenkuan;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ItmySpringbootDatasourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ItmySpringbootDatasourceApplication.class, args);
    }

}

```



**以上看itmy-springboot-datasource这个项目。**



考虑的点：？？？

现在我test01里面的服务要调test02库中的服务。这时候怎么控制事务？？因为有两个事务管理器，该指定哪一个？

![1570158525676](.\assets\1570158525676.png)

具体看下一篇：SpringBoot的事物管理，也附带项目。在该项目基础上修改。







No qualifying bean of type [javax.sql.DataSource] is defined: expected single matching bean but found 2: test1DataSource,test2DataSource

加上@Primary即可。

There was an unexpected error (type=Internal Server Error, status=500).

No qualifying bean of type 'org.springframework.transaction.PlatformTransactionManager' available: expected single matching bean but found 2: test1TransactionManager,test2TransactionManager

 

 指定事务管理器



Springboot1.5的时候没有默认指向数据源 会报错

Springboot2.0的时候不报错