## 一、SpringBoot启动方式1

```java
package com.nieshenkuan.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Description:SpringBoot应用启动的第一种方式
 * 在类上加@EnableAutoConfiguration注解开启自动配置。但未开启扫包模式。所以有局限性。
 * 然后在该类中定义一个main方法。用 SpringApplication.run(StartController.class, args);启动应用。
 * @author: nieshenkuan
 * @Date: 2019/10/1 13:05
 */

@RestController
@EnableAutoConfiguration
public class StartController {

    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }
    public static void main(String[] args) {
        SpringApplication.run(StartController.class, args);
    }
}
```



`@EnableAutoConfiguration`:注解:作用在于让 Spring Boot 根据应用所声明的依赖来对 Spring 框架进行自动配置;
这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置Spring。由于spring-boot-starter-web添加了Tomcat和Spring MVC，所以auto-configuration将假定你正在开发一个web应用并相应地对Spring进行设置。

## 二、SpringBoot启动方式2
```java
package com.nieshenkuan;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

/**
 * @Description:SpringBoot应用启动的第二中方式
 * 在第一种方式上面，加上了包扫描功能。这样我们可以指定扫描的包的位置。@ComponentScan(basePackages = "com.nieshenkuan.controller")
 * @author: nieshenkuan
 * @Date: 2019/10/1 13:13
 */
@ComponentScan(basePackages = "com.nieshenkuan.controller")
@EnableAutoConfiguration
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}
```

## 三、SpringBoot启动方式3
```java
/**
 *@Description: Springboot应用启动的第三种方式。通过@SpringBootApplication注解
 * 通过@SpringBootApplication注解启动springboot应用。
 * 这个注解是@EnableAutoConfiguration+@ComponentScan的组合。既开启了自动配置，也开启了包扫描。
 * 但是包扫描默认是当前类包下的所有包及其子包。
 * 这种方式也是用的最多的。推荐用这个。
 *@Author: nieshenkuan
 *@Date: 2019/10/1 13:19
 */
@SpringBootApplication
public class ItmySpringbootStartmodeApplication {

    
    public static void main(String[] args) {
        SpringApplication.run(ItmySpringbootStartmodeApplication.class, args);
    }
}
```



`@SpringBootApplication`:

@SpringBootApplication 被 @Configuration、@EnableAutoConfiguration、@ComponentScan 注解所修饰，换言之 Springboot 提供了统一的注解来替代以上三个注解;

扫包范围：在启动类上加上@SpringBootApplication注解,当前包下或者子包下所有的类都可以扫到。