# **一、**Spring Boot 入门

## 1、Spring Boot 简介

> 简化Spring应用开发的一个框架；
>
> 整个Spring技术栈的一个大整合；
>
> J2EE开发的一站式解决方案；

Spring Boot来简化Spring应用开发，约定大于配置，去繁从简，just run就能创建一个独立的，产品级别的。

**应用背景：**
J2EE笨重的开发、繁多的配置、低下的开发效率、
复杂的部署流程、第三方技术集成难度大。
**解决：**
“Spring全家桶”时代。
Spring Boot  J2EE一站式解决方案
Spring Cloud 分布式整体解决方案



**优点：**
– 快速创建独立运行的Spring项目以及与主流框架集成
– 使用嵌入式的Servlet容器，应用无需打成WAR包
– starters自动依赖与版本控制
– 大量的自动配置，简化开发，也可修改默认值
– 无需配置XML，无代码生成，开箱即用
– 准生产环境的运行时应用监控
– 与云计算的天然集成





## 2、微服务

2014，martin fowler

==微服务：架构风格（服务微化）==

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

![1560955895723](/assets/1560955895723.png)



单体应用：ALL IN ONE

![1560955944222](.\assets\1560955944222.png)

**微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；**

![1561815991653](.\assets\1561815991653.png)

**[详细参照微服务文档**](https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa)



## 3、环境准备

环境约束

–jdk1.8：Spring Boot 推荐jdk1.7及以上；java version "1.8.0_112"

–maven3.x：maven 3.3以上版本；Apache Maven 3.3.9

–IntelliJIDEA2017：IntelliJ IDEA 2017.2.2 x64、STS

–SpringBoot 1.5.9.RELEASE：1.5.9；

统一环境；



### 1、MAVEN设置；

给maven 的settings.xml配置文件的profiles标签添加：告诉maven用jdk1.8编译，打包等等。蛮重要的。

```xml
<profile>
  <id>jdk-1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

### 2、IDEA设置

整合maven进来；

![1560956035754](.\assets\1560956035754.png)



![1560956062808](.\assets\1560956062808.png)

## 4、Spring Boot HelloWorld

一个功能：

浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串；



### 1、创建一个maven工程；（jar）

![1560956128951](.\assets\1560956128951.png)

![1560956205365](.\assets\1560956205365.png)



![1560956301821](.\assets\1560956301821.png)





![1560956342877](.\assets\1560956342877.png)



![1560956425853](.\assets\1560956425853.png)







### 2、在pom.xml中导入spring boot相关的依赖

【具体可以看官网上的springboot的quickstart】

**SpringBoot项目都有一个共同的父项目：spring-boot-starter-parent**

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```



### 3、编写一个主程序；启动Spring Boot应用

```java

/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```



**@SpringBootApplication**： @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用





### 4、编写相关的Controller、Service

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```





### 5、运行主程序测试

### 6、简化部署

【pom.xml文件中加入这个插件】

```xml
 <!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```



【idea中打成jar包步骤：】

![1560957115587](.\assets\1560957115587.png)

==将这个应用打成jar包，直接使用**java -jar**的命令进行执行；==boot项目以后都会打成一个jar包，不会再打成传统的war包。





## 5、Hello World探究

### 1、POM文件

#### 1、父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>

他的父项目是
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>1.5.9.RELEASE</version>
  <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
他来真正管理Spring Boot应用里面的所有依赖版本；
```

Spring Boot的版本仲裁中心：spring-boot-dependencies

以后我们导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖自然需要声明版本号）

---

每一个springboot项目都有一个父项目：spring-boot-starter-parent；同时这个spring-boot-starter-parent又有一个spring-boot-dependencies的父项目，这个父项目中指定了一些jar包的版本。没有指定的jar包的版本需要手动指定。





#### 2、启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```





##### 看一下这个spring-boot-starter-web依赖里面依赖了哪些？

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starters</artifactId>
		<version>1.5.9.RELEASE</version>
	</parent>
	<artifactId>spring-boot-starter-web</artifactId>
	<name>Spring Boot Web Starter</name>
	<description>Starter for building web, including RESTful, applications using Spring
		MVC. Uses Tomcat as the default embedded container</description>
	<url>http://projects.spring.io/spring-boot/</url>
	<organization>
		<name>Pivotal Software, Inc.</name>
		<url>http://www.spring.io</url>
	</organization>
	<properties>
		<main.basedir>${basedir}/../..</main.basedir>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
		</dependency>
	</dependencies>
</project>

```





##### spring-boot-starter-==web==：

> spring-boot-starter：spring-boot场景启动器；帮我们导入了web模块正常运行所依赖的组件；

> Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。





### 2、主程序类，主入口类【重要】

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

        // Spring应用启动起来
       SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}

```



#### （1）、@SpringBootApplication注解

> @**SpringBootApplication**:    Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；





#### （2）、SpringBootApplication注解源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class,
        attribute = "exclude"
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class,
        attribute = "excludeName"
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```





#### （3）、@**SpringBootConfiguration**:Spring Boot的配置类；

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    
}
```

标注在某个类上，表示这是一个Spring Boot的配置类；

`@Configuration`:配置类上来标注这个注解；`@Configuration`就是替换以前的xml配置文件的。

**配置类 == 配置文件**；配置类也是容器中的一个组件；上面也有`@Component`注解





#### （4）、@Configuration注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```



#### （5）、@**EnableAutoConfiguration**

@**EnableAutoConfiguration**：开启自动配置功能；**Spring Boot的核心注解**，重要。

以前我们需要配置的东西，Spring Boot帮我们自动配置；@**EnableAutoConfiguration**告诉SpringBoot开启自动配置功能；这样自动配置才能生效；

`@EnableAutoConfiguration`源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```



#### （6）、@**AutoConfigurationPackage**

@**AutoConfigurationPackage**：自动配置包；

@AutoConfigurationPackage的含义是将主配置类所在的包的所有的组件都扫描进去。

==将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；==

![1561819365838](.\assets\1561819365838.png)





#### （7）、@**Import**(AutoConfigurationPackages.Registrar.class)：

@**Import**(AutoConfigurationPackages.Registrar.class)：

Spring的底层注解`@Import`，给容器中导入一个组件；导入的组件由AutoConfigurationPackages.Registrar.class；





@**Import**(EnableAutoConfigurationImportSelector.class)；
给容器中导入组件？
**EnableAutoConfigurationImportSelector**：导入哪些组件的选择器；
将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；
会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；	**EnableAutoConfigurationImportSelector**这个java类中定义了一个configurations变量，可以debug进来看一下。

	

![1560958633485](.\assets\1560958633485.png)

![企业微信截图_15609927728763](.\assets\企业微信截图_15609927728763.png)



有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

这些都是在`AutoConfigurationImportSelector`这个类中的。

	SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader)；

```
  protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```





==Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；==以前我们需要自己配置的东西，自动配置类都帮我们；

![1560993562609](.\assets\1560993562609.png)

**J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-1.5.9.RELEASE.jar；**



		

### 3、HelloWorld总结

#### 1、starters

* Spring Boot为我们提供了简化企业级开发绝大多数场景的starter pom（启动
  器），只要引入了相应场景的starter pom，相关技术的绝大部分配置将会消
  除（自动配置），从而简化我们开发。业务中我们就会使用到Spring Boot为
  我们自动配置的bean

* 参考 https://docs.spring.io/springboot/docs/1.5.9.RELEASE/reference/htmlsingle/#using-boot-starter

* 这些starters几乎涵盖了javaee所有常用场景，Spring Boot对这些场景依赖的jar也做了严格的测试与版本控制。我们不必担心jar版本合适度问题。

* spring-boot-dependencies里面定义了jar包的版本


#### 2、入口类和@SpringBootApplication

* 1、程序从main方法开始运行
* 2、使用SpringApplication.run()加载主程序类
* 3、主程序类需要标注@SpringBootApplication
* 4、 @EnableAutoConfiguration是核心注解；
* 5、@Import导入所有的自动配置场景
* 6、@AutoConfigurationPackage定义默认的包扫描规则
* 7、程序启动扫描加载主程序类所在的包以及下面所有子包的组件；

![1561822510690](.\assets\1561822510690.png)



#### 3、自动配置

1、xxxAutoConfiguration
– Spring Boot中存现大量的这些类，这些类的作用就是帮我们进行自动配置
– 他会将这个这个场景需要的所有组件都注册到容器中，并配置好
– 他们在类路径下的 META-INF/spring.factories文件中
– spring-boot-autoconfigure-1.5.9.RELEASE.jar中包含了所有场景的自动
配置类代码
– 这些自动配置类是Spring Boot进行自动配置的精髓



## 6、使用Spring Initializer快速创建Spring Boot项目

### 1、IDEA：使用 Spring Initializer快速创建项目

IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；

选择我们需要的模块；向导会联网创建Spring Boot项目；

默认生成的Spring Boot项目；

- 主程序已经生成好了，我们只需要我们自己的逻辑
- resources文件夹中目录结构
  - static：保存所有的静态资源； js css  images；
  - templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）；
  - application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；

### 2、STS使用 Spring Starter Project快速创建项目

![1560959125837](.\assets\1560959125837.png)



![1560959198476](.\assets\1560959198476.png)



![1560959324441](.\assets\1560959324441.png)



![1560959349349](.\assets\1560959349349.png)

![1560959374238](.\assets\1560959374238.png)

[选择你要的模块，比如这里选择web开发模块。这里面就勾选上web模块。]

