# 十六、Spring Boot与监控管理

## 监控管理

##  一、Actuator监控应用

> Actuator是spring boot的一个附加功能,可帮助你在应用程序生产环境时监视和管理应用程序。可以使用HTTP的各种请求来监管,审计,收集应用的运行情况.特别对于微服务管理十分有意义.缺点：没有可视化界面。

![1570109618445](.\assets\1570109618445.png)

通过引入spring-boot-starter-actuator，可以使用Spring Boot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计、健康及指标信息等。

**步骤：**
引入spring-boot-starter-actuator
通过http方式访问监控端点
可进行shutdown（POST 提交，此端点默认关闭）


**监控和管理端点**

| 端点名      | 描述                        |
| ----------- | --------------------------- |
| autoconfig  | 所有自动配置信息            |
| auditevents | 审计事件                    |
| beans       | 所有Bean的信息              |
| configprops | 所有配置属性                |
| dump        | 线程状态信息                |
| env         | 当前环境信息                |
| health      | 应用健康状况                |
| info        | 当前应用信息                |
| metrics     | 应用的各项指标              |
| mappings    | 应用@RequestMapping映射路径 |
| shutdown    | 关闭当前应用（默认关闭）    |
| trace       | 追踪信息（最新的http请求）  |



## 二、定制端点信息

* 定制端点一般通过**endpoints+端点名+属性名**来设置。
* 修改端点id（endpoints.beans.id=mybeans）
* 开启远程应用关闭功能（endpoints.shutdown.enabled=true）
* 关闭端点（endpoints.beans.enabled=false）
* 开启所需端点
  （1）、endpoints.enabled=false
  （2）、endpoints.beans.enabled=true
* 定制端点访问根路径
  management.context-path=/manage
* 关闭http端点
  management.port=-1

## 三、工程示例



### pom.xml

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

核心要导入监控的依赖



### 开启监控

```properties
management.security.enabled=false
```



### 浏览器访问监控信息





**注：看springboot-actuator项目。**



