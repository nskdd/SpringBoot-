#  性能优化

## 1、组件自动扫描带来的问题



默认情况下，我们会使用 @SpringBootApplication 注解来自动获取应用的配置信息，但这样也会给应用带来一些副作用。使用这个注解后，会触发自动配置（ auto-configuration ）和 组件扫描 （ component scanning ），这跟使用 @Configuration、@EnableAutoConfiguration 和 @ComponentScan 三个注解的作用是一样的。这样做给开发带来方便的同时，也会有三方面的影响：

1、会导致项目启动时间变长。当启动一个大的应用程序,或将做大量的集成测试启动应用程序时，影响会特别明显。

2、会加载一些不需要的多余的实例（beans）。

3、会增加 CPU 消耗。

针对以上三个情况，我们可以移除 @SpringBootApplication 和 @ComponentScan 两个注解来禁用组件自动扫描，然后在我们需要的 bean 上进行显式配置：

```java
//// 移除 @SpringBootApplication and @ComponentScan, 用 @EnableAutoConfiguration 来替代
//@SpringBootApplication
@Configuration
@EnableAutoConfiguration
public class App01 {

	public static void main(String[] args) {
		SpringApplication.run(App01.class, args);
	}

}
```



##  2、**将Servlet容器变成Undertow**

默认情况下，Spring Boot 使用 Tomcat 来作为内嵌的 Servlet 容器

可以将 Web 服务器切换到 Undertow 来提高应用性能。Undertow 是一个采用 Java 开发的灵活的高性能 Web 服务器，提供包括阻塞和基于 NIO 的非堵塞机制。Undertow 是红帽公司的开源产品，是 Wildfly 默认的 Web 服务器。首先，从依赖信息里移除 Tomcat 配置：

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-tomcat</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```



然后添加 Undertow：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```



##  3、SpringBoot JVM参数调优

这个根据服务器的内存大小，来设置堆参数。

-Xms :设置Java堆栈的初始化大小

-Xmx :设置最大的java堆大小

实例参数-XX:+PrintGCDetails -Xmx32M -Xms1M

本地项目调优

![img](.\assets\wps1.jpg) 

外部运行调优

java -server -Xms32m -Xmx32m  -jar springboot_v2.jar

 

利用jconsole来查看堆内存分配情况。Jdk的bin里面。

t