#  2.0版本新特性

## 1、以Java 8 为基准

Spring Boot 2.0 要求Java 版本必须8以上， Java 6 和 7 不再支持。

##  2、内嵌容器包结构调整

为了支持reactive使用场景，内嵌的容器包结构被重构了的幅度有点大。EmbeddedServletContainer被重命名为WebServer，并且org.springframework.boot.context.embedded 包被重定向到了org.springframework.boot.web.embedded包下。举个例子，如果你要使用TomcatEmbeddedServletContainerFactory回调接口来自定义内嵌Tomcat容器，你现在应该使用TomcatServletWebServerFactory。



## 3、Servlet-specific 的server properties调整

大量的Servlet专属的server.* properties被移到了server.servlet下：

| Old property                 | New property                         |
| ---------------------------- | ------------------------------------ |
| server.context-parameters.*  | server.servlet.context-parameters.*  |
| server.context-path          | server.servlet.context-path          |
| server.jsp.class-name        | server.servlet.jsp.class-name        |
| server.jsp.init-parameters.* | server.servlet.jsp.init-parameters.* |
| server.jsp.registered        | server.servlet.jsp.registered        |
| server.servlet-path          | server.servlet.path                  |



由此可以看出一些端倪，那就是server不再是只有servlet了，还有其他的要加入。



## 4、Actuator 默认映射

actuator的端点（endpoint）现在默认映射到/application，比如，/info 端点现在就是在/application/info。但你可以使用management.context-path来覆盖此默认值。



## 5、Spring Loaded不再支持

由于Spring Loaded项目已被移到了attic了，所以不再支持Spring Loaded了。现在建议你去使用Devtools。Spring Loaded不再支持了。



## 6、支持Quartz Scheduler



Spring Boot 2 针对Quartz调度器提供了支持。你可以加入spring-boot-starter-quartz starter来启用。而且支持基于内存和基于jdbc两种存储。



##  7、OAuth 2.0 支持

Spring Security OAuth 项目中的功能将会迁移到Spring Security中。将会OAuth 2.0。



##  8、支持Spring WebFlux

WebFlux 模块的名称是 spring-webflux，名称中的 Flux 来源于 Reactor 中的类 Flux。该模块中包含了对反应式 HTTP、服务器推送事件和 WebSocket 的客户端和服务器端的支持。对于开发人员来说，比较重要的是服务器端的开发，这也是本文的重点。在服务器端，WebFlux 支持两种不同的编程模型：第一种是 Spring MVC 中使用的基于 Java 注解的方式；第二种是基于 Java 8 的 lambda 表达式的函数式编程模型。这两种编程模型只是在代码编写方式上存在不同。它们运行在同样的反应式底层架构之上，因此在运行时是相同的。WebFlux 需要底层提供运行时的支持，WebFlux 可以运行在支持 Servlet 3.1 非阻塞 IO API 的 Servlet 容器上，或是其他异步运行时环境，如 Netty 和 Undertow。



## 9、**版本要求**

Jetty

要求Jetty最低版本为9.4。

 

Tomcat

要求Tomcat最低版本为8.5。

 

Hibernate

要求Hibernate最低版本为5.2。

 

Gradle

要求Gradle最低版本为3.4。

 

SendGrid

SendGrid最低支持版本是3.2。为了支持这次升级，username和password已经被干掉了。因为API key现在是唯一支持的认证方式。





