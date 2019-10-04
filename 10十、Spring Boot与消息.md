# 十、Spring Boot与消息

（JMS、AMQP、RabbitMQ）

## 一、概述

#### 1、大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力
#### 2、消息服务中两个重要概念：
**==消息代理（message broker）和目的地（destination==**
**当消息发送者发送消息以后，将由消息代理（可以理解为消息队列服务器）接管，消息代理保证消息传递到指定目的地。**

#### 3、消息队列主要有两种形式的目的地
##### （1）、队列（queue）：点对点消息通信（point-to-point） 
##### （2）、主题（topic）：发布（publish）/订阅（subscribe）消息通信



#### 消息队列的几种应用场景

##### 场景一：消息队列-异步消息



![1561961954080](.\assets\1561961954080.png)

消息队列-异步消息：以上在注册信息写入到数据库之后，要进行两个操作然后给客户端响应；

通过消息队列，将这两个操作写入到消息队列中（写入到消息队列中速度很快），然后异步读取消息队列中的数据，执行。



##### 场景二：消息队列-应用解耦

![1564499006661](.\assets\1564499006661.png)

##### 场景三：消息队列-流量削峰

![1564499159495](.\assets\1564499159495.png)





#### 4、点对点式：
消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列
消息只有唯一的发送者和接受者，但并不是说只能有一个接收者

#### 5、发布订阅式：
发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

#### 6、JMS（Java Message Service）JAVA消息服务：
基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

#### 7、AMQP（Advanced Message Queuing Protocol）
高级消息队列协议，也是一个消息代理的规范，兼容JMS
RabbitMQ是AMQP的实现



**JMS与AMQP两种规范对比：**

![1561962609684](.\assets\1561962609684.png)



#### 8、Spring对两种规范的支持支持
* spring-jms提供了对JMS的支持
* spring-rabbit提供了对AMQP的支持
* 需要ConnectionFactory的实现来连接消息代理
* 提供JmsTemplate、RabbitTemplate来发送消息
* @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
* @EnableJms、@EnableRabbit开启支持

#### 9、Spring Boot自动配置
* JmsAutoConfiguration
* RabbitAutoConfiguration

注意一下以上两个自动配置类。



## 二、RabbitMQ简介

#### RabbitMQ简介：
RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。

#### 核心概念
##### Message
消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

##### Publisher
消息的生产者，也是一个向交换器发布消息的客户端应用程序。

##### Exchange
交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别

##### Queue
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

##### Binding
绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
Exchange 和Queue的绑定可以是多对多的关系。

##### Connection
网络连接，比如一个TCP连接。

##### Channel
信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

##### Consumer
消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

##### Virtual Host
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

##### Broker
表示消息队列服务器实体



##### 以上核心概念的位置图

![1561962695677](.\assets\1561962695677.png)



## 三、RabbitMQ运行机制

#### AMQP 中的消息路由
AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 **Exchange 和 Binding** 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![1561962725332](.\assets\1561962725332.png)

#### **Exchange 类型**

Exchange分发消息时根据类型的不同分发策略有区别，**目前共四种类型：direct、fanout、topic、headers** 。headers 匹配 AMQP 消息的 header 而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：



##### 第一种：

![1561962756981](.\assets\1561962756981.png)

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。



##### 第二种：

![1561963032549](.\assets\1561963032549.png)

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。



##### 第三种：

![1561963052433](.\assets\1561963052433.png)

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#匹配0个或多个单词，*匹配一个单词。



## 四、RabbitMQ整合

1、引入 spring-boot-starter-amqp
2、application.yml配置
3、测试RabbitMQ
（1）、AmqpAdmin：管理组件
（2）、RabbitTemplate：消息发送处理组件

![1561963118194](.\assets\1561963118194.png)

rabbitmq需要安装。这里暂时只是过一遍，并没有安装。

amqp的核心即交换器、队列、绑定器。消息先到交换器，交换器根据绑定关系（这里是交换器会跟一个队列绑定，或多个队列。根据绑定器），然后消息到消息队列（可以理解为一个消息服务器）



首先应该安装rabbitmq，就跟redis类似。然后通过ip连接到rabbitmq的服务器。然后连上服务器创建相应的交换器、队列、绑定器等等。当然，也可以再代码中创建以上组件。

假设我已经安装了rabbitmq，这里创建项目如下：

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.atguigu</groupId>
	<artifactId>springboot-02-amqp</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springboot-02-amqp</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.12.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
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



#### rabbitmq基本操作

rabbitmq给我们提供了一个操作的队列的对象RabbitTemplate。

简单基本操作如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot02AmqpApplicationTests {

	@Autowired
	RabbitTemplate rabbitTemplate;
		/**
	 * 1、单播（点对点）
	 */
	@Test
	public void contextLoads() {
		//Message需要自己构造一个;定义消息体内容和消息头
		//rabbitTemplate.send(exchage,routeKey,message);

		//object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq；
		//rabbitTemplate.convertAndSend(exchage,routeKey,object);
		Map<String,Object> map = new HashMap<>();
		map.put("msg","这是第一个消息");
		map.put("data", Arrays.asList("helloworld",123,true));
		//对象被默认序列化以后发送出去
		rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",new Book("西游记","吴承恩"));

	}

	//接受数据,如何将数据自动的转为json发送出去
	@Test
	public void receive(){
		Object o = rabbitTemplate.receiveAndConvert("atguigu.news");
		System.out.println(o.getClass());
		System.out.println(o);
	}

	/**
	 * 广播
	 */
	@Test
	public void sendMsg(){
		rabbitTemplate.convertAndSend("exchange.fanout","",new Book("红楼梦","曹雪芹"));
	}

}
```

一般的话，发送到队列中的数据是走jdk的序列化，用的是字节数组。我们可以修改默认的序列化机制，一般我们利用json来序列化。操作如下：替换原来的messagecovert；

```java
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyAMQPConfig {

    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```



我们发送给消息队列后，我们这边如何自动获取到队列里面的信息呢？Spring给我们提供了一个相关的监听器，能够监听相对应的队列中的消息是否增加，然后进行何种操作。如下步骤：

（1）、在主配置类上开启基于注解的RabbitMQ模式:@EnableRabbit

```java
/**
 * 自动配置
 *  1、RabbitAutoConfiguration
 *  2、有自动配置了连接工厂ConnectionFactory；
 *  3、RabbitProperties 封装了 RabbitMQ的配置
 *  4、 RabbitTemplate ：给RabbitMQ发送和接受消息；
 *  5、 AmqpAdmin ： RabbitMQ系统管理功能组件;
 *  	AmqpAdmin：创建和删除 Queue，Exchange，Binding
 *  6、@EnableRabbit +  @RabbitListener 监听消息队列的内容
 *
 */
@EnableRabbit  //开启基于注解的RabbitMQ模式
@SpringBootApplication
public class Springboot02AmqpApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot02AmqpApplication.class, args);
	}
}
```



(2)、利用@RabbitListener注解监听指定队列里面的消息。

```java
import com.atguigu.amqp.bean.Book;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

@Service
public class BookService {

    @RabbitListener(queues = "atguigu.news")
    public void receive(Book book){
        System.out.println("收到消息："+book);
    }

    @RabbitListener(queues = "atguigu")
    public void receive02(Message message){
        System.out.println(message.getBody());
        System.out.println(message.getMessageProperties());
    }
}
```



附上Book.java

```java


public class Book {
    private String bookName;
    private String author;

    public Book() {
    }

    public Book(String bookName, String author) {
        this.bookName = bookName;
        this.author = author;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    @Override
    public String toString() {
        return "Book{" +
                "bookName='" + bookName + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
}

```

这个测试要启动整个服务。



我们除了在客户端能够创建和删除队列、绑定器等等组件，在代码中可以也可以操作，主要是通过这个对象来操作的：AmqpAdmin ： RabbitMQ系统管理功能组件;

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot02AmqpApplicationTests {

	@Autowired
	RabbitTemplate rabbitTemplate;

	@Autowired
	AmqpAdmin amqpAdmin;

	@Test
	public void createExchange(){

//		amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
//		System.out.println("创建完成");

//		amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
		//创建绑定规则

//		amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqp.haha",null));

		//amqpAdmin.de
	}
		}
}
```

