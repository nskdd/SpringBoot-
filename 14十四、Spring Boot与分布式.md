# 十四、Spring Boot与分布式

（分步式、Dubbo/Zookeeper、Spring Boot/Cloud）

## 一、分布式应用

在分布式系统中，国内常用zookeeper+dubbo组合，而Spring Boot推荐使用全栈的Spring，Spring Boot+Spring Cloud。

分布式系统：

![1561963689180](.\assets\1561963689180.png)

![1561963702613](.\assets\1561963702613.png)

## 二、Zookeeper和Dubbo

![1561963724213](.\assets\1561963724213.png)

![1561963735247](.\assets\1561963735247.png)

![1561963744631](.\assets\1561963744631.png)

## 三、Zookeeper+Dubbo整合示例

### 3.1、安装Zookeeper作为注册中心

我是在本地安装的zookeeper，启动zookeeper服务即可。

### 3.2、创建服务提供者

新建一个空工程springboot-dubbo，然后在空工程里面新建一个modul，名为provider-ticket。

#### pom文件

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>com.alibaba.boot</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>0.1.0</version>
		</dependency>

		<!--引入zookeeper的客户端工具-->
		<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>


		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

```

#### 编写服务提供者（接口跟接口实现类）

```java
package com.nieshenkuan.ticket.service;

/**
 * @Description: 服务提供者接口
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:16
 */
public interface TicketService {

    public String getTicket();
}

```

```java
package com.nieshenkuan.ticket.service;
import com.alibaba.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;
/**
 * @Description: 服务提供者的实现
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:17
 */

@Component
@Service  //将服务发布出去，注意要用com.alibaba.dubbo.config.annotation.Service;的注解
public class TicketServiceImpl implements TicketService {
    @Override
    public String getTicket() {
        return "《厉害了，我的国》";
    }
}
```

#### 将服务注册到注册中心

```properties
dubbo.application.name=provider-ticket
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.scan.base-packages=com.nieshenkuan.ticket.service
```



### 3.3、编写服务消费者

在空工程里面新建一个modul，名为consumer-user。

#### pom文件

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>com.alibaba.boot</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>0.1.0</version>
		</dependency>

		<!--引入zookeeper的客户端工具-->
		<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>


		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

#### 编写服务，跟服务提供者方的包名目录结构以及服务名称要一致



**TicketService接口**

```java
package com.nieshenkuan.ticket.service;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:19
 */
public interface TicketService {

    public String getTicket();
}
```

并没有实现这个接口。

#### 使用接口，但具体调用的是消费提供者中的接口

```java
package com.nieshenkuan.user.service;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:20
 */
import com.alibaba.dubbo.config.annotation.Reference;
import com.nieshenkuan.ticket.service.TicketService;
import org.springframework.stereotype.Service;

@Service
public class UserService{

    @Reference
    TicketService ticketService;

    public void hello(){
        String ticket = ticketService.getTicket();
        System.out.println("买到票了："+ticket);
    }


}
```

@Reference注解，是com.alibaba.dubbo.config.annotation.Reference中的注解。

#### 配置信息

```properties
dubbo.application.name=consumer-user
dubbo.registry.address=zookeeper://127.0.0.1:2181
```



### 3.4、整合dubbo（dubbo，rpc框架，来进行服务之间的通信）



#### 服务提供方

##### 服务类

```java
package com.nieshenkuan.ticket.service;

/**
 * @Description: 服务提供者接口
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:16
 */
public interface TicketService {

    public String getTicket();
}

```

```java
package com.nieshenkuan.ticket.service;
import com.alibaba.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;
/**
 * @Description: 服务提供者的实现
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:17
 */

@Component
@Service  //将服务发布出去，注意要用com.alibaba.dubbo.config.annotation.Service;的注解
public class TicketServiceImpl implements TicketService {
    @Override
    public String getTicket() {
        return "《厉害了，我的国》";
    }
}
```



##### 配置信息

```properties
dubbo.application.name=provider-ticket
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.scan.base-packages=com.nieshenkuan.ticket.service
```



#### 服务消费者

##### 编写跟服务提供方一样包结构的服务的接口与方法，但没有实现类

```java
package com.nieshenkuan.ticket.service;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:19
 */
public interface TicketService {

    public String getTicket();
}
```

注意包名，跟上面服务提供方的TicketService接口的一致。很重要。



##### 通过@Reference注解获取服务提供者中的的接口

```java
package com.nieshenkuan.user.service;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/8/3 17:20
 */
import com.alibaba.dubbo.config.annotation.Reference;
import com.nieshenkuan.ticket.service.TicketService;
import org.springframework.stereotype.Service;

@Service
public class UserService{

    @Reference
    TicketService ticketService;

    public void hello(){
        String ticket = ticketService.getTicket();
        System.out.println("买到票了："+ticket);
    }


}
```



##### 配置类

```properties
dubbo.application.name=consumer-user
dubbo.registry.address=zookeeper://127.0.0.1:2181
```



**springboot-dubbo项目**



## 四、Spring Boot和Spring Cloud

![1561963777095](.\assets\1561963777095.png)



微服务

![1561963867935](.\assets\1561963867935.png)

Martin Fowler 微服务原文 https://martinfowler.com/articles/microservices.html  





Spring Cloud 入门
1、创建provider
2、创建consumer
3、引入Spring Cloud
4、引入Eureka注册中心
5、引入Ribbon进行客户端负载均衡



## 五、SpringBoot与SpringCloud整合示例

### 5.1、创建Eureka注册中心

#### 1、pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.nieshenkaun</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-server</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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



#### 2、配置Eureka信息

```yml
server:
  port: 8761
eureka:
  instance:
    hostname: eureka-server  # eureka实例的主机名
  client:
    register-with-eureka: false #不把自己注册到eureka上
    fetch-registry: false #不从eureka上来获取服务的注册信息
    service-url:
      defaultZone: http://localhost:8761/eureka/

```



#### 3、主配置类上加@EnableEurekaServer

```java
/**
 * 注册中心
 * 1、配置Eureka信息
 * 2、@EnableEurekaServer
 */
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}

```



### 5.2、创建provider



##### 1、pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.nieshenkuan</groupId>
    <artifactId>provider-ticket</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>provider-ticket</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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



##### 2、编写服务类

```java
package com.nieshenkuan.service;

import org.springframework.stereotype.Service;

/**
 * @Description: 服务提供者
 * @author: nieshenkuan
 * @Date: 2019/8/4 15:12
 */
@Service
public class TicketService {

    public String getTicket(){
        System.out.println("8002");
        return "《厉害了，我的国》";
    }
}

```



```java
package com.nieshenkuan.controller;

import com.nieshenkuan.service.TicketService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/8/4 15:13
 */
@RestController
public class TicketController {

    @Autowired
    TicketService ticketService;

    @GetMapping("/ticket")
    public String getTicket(){
        return ticketService.getTicket();
    }
}

```





##### 3、配置文件application.yml

```yml
server:
  port: 8002
spring:
  application:
    name: provider-ticket
eureka:
  instance:
    prefer-ip-address: true # 注册服务的时候使用服务的ip地址
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```





### 5.3、创建consumer

#### 1、pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.nieshenkuan</groupId>
    <artifactId>consumer-user</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>consumer-user</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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



#### 2、添加RestTemplate组件到容器中

```java
package com.nieshenkuan;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

/**
 *
 */
@EnableDiscoveryClient //开启发现服务功能
@SpringBootApplication
public class ConsumerUserApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerUserApplication.class, args);
    }

    @LoadBalanced //使用负载均衡机制
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}

```



#### 3、@EnableDiscoveryClient //开启发现服务功能

```java
package com.nieshenkuan;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

/**
 *
 */
@EnableDiscoveryClient //开启发现服务功能
@SpringBootApplication
public class ConsumerUserApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerUserApplication.class, args);
    }

    @LoadBalanced //使用负载均衡机制
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}

```



#### 4、调用服务提供方的方法，通过RestTemplate

```java
package com.nieshenkuan.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/8/4 15:22
 */
@RestController
public class UserController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/buy")
    public String buyTicket(String name){
        String s = restTemplate.getForObject("http://PROVIDER-TICKET/ticket", String.class);
        return name+"购买了"+s;
    }
}

```



#### 5、配置文件

```yml
spring:
  application:
    name: consumer-user
server:
  port: 8200

eureka:
  instance:
    prefer-ip-address: true # 注册服务的时候使用服务的ip地址
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```



**springboot-springcloud项目**

