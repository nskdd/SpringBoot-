# Mybatis整合分页插件

## 1、pageHelper

> PageHelper 是一款好用的开源免费的 Mybatis 第三方物理分页插件
>
> 物理分页
>
> 支持常见的 12 种数据库。Oracle,MySql,MariaDB,SQLite,DB2,PostgreSQL,SqlServer 等
>
> 支持多种分页方式
>
> 支持常见的 RowBounds(PageRowBounds)，PageHelper.startPage 方法调用，Mapper 接口参数调用



## 2、Maven依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.nieshenkuan</groupId>
    <artifactId>itmy-springboot-mybatis-pagehelper</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>itmy-springboot-mybatis-pagehelper</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- springboot 整合 pagehelper -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```



## 3、配置文件

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/itmy_test01?useUnicode=true&characterEncoding=UTF-8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver


logging.level.com.nieshenkuan.mapper=DEBUG
pagehelper.helperDialect=mysql
pagehelper.reasonable=true
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql
pagehelper.page-size-zero=true


```



##  4、Entity层



```java
package com.nieshenkuan.entity;

import lombok.Data;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/2 13:35
 */
@Data
public class User {

    private Integer id;
    private String name;
    private Integer age;
}

```



##  5、Mapper层

```java
package com.nieshenkuan.mapper;

import com.nieshenkuan.entity.User;
import org.apache.ibatis.annotations.Select;

import java.util.List;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/2 13:36
 */
public interface UserMapper {
    @Select("SELECT * FROM USER  ")
    List<User> findUserList();
}

```





## 6、Service层

```java
package com.nieshenkuan.service;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.nieshenkuan.entity.User;
import com.nieshenkuan.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/2 13:37
 */
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    /**
     * page 当前页数<br>
     * pageSize 当前展示多少个<br>
     * @param page
     * @param pageSize
     * @return
     */
    // 思路
    public PageInfo<User> findUserList(int page, int pageSize) {
        // mysql 查询 limit  oraclet 伪列   sqlserver top
        // pageHelper 帮我们生成分页语句
        PageHelper.startPage(page, pageSize); // 底层实现原理采用改写语句
        List<User> listUser = userMapper.findUserList();
        //	PageInfo<User> pageInfoUserList= userMapper.findUserList();
        // 返回给客户端展示
        PageInfo<User> pageInfoUserList = new PageInfo<User>(listUser);
        // 实现原理
        return pageInfoUserList;
    }

}

```



##  7、Controller层

```java
package com.nieshenkuan.controller;
import com.nieshenkuan.entity.User;
import com.nieshenkuan.service.UserService;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.github.pagehelper.PageInfo;
/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/2 13:36
 */
@RestController
public class IndexController {

    @Autowired
    private UserService userService;

    @RequestMapping("/index")
    public PageInfo<User> index(int page, int pageSize) {
        return userService.findUserList(page, pageSize);
    }

}

```



## 8、启动项目

```java
package com.nieshenkuan;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@MapperScan("com.nieshenkuan.mapper")
@SpringBootApplication
public class ItmySpringbootMybatisPagehelperApplication {

    public static void main(String[] args) {
        SpringApplication.run(ItmySpringbootMybatisPagehelperApplication.class, args);
    }

}

```







**详细看itmy-springboot-mybatis-pagehelper项目。**

