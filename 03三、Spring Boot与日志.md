# 三、日志

## 1、日志框架

 小张；开发一个大型系统；

	1、System.out.println("")；将关键数据打印在控制台；去掉？写在一个文件？
	2、框架来记录系统的一些运行时信息；日志框架 ；  zhanglogging.jar；
	3、高大上的几个功能？异步模式？自动归档？xxxx？  zhanglogging-good.jar？
	4、将以前框架卸下来？换上新的框架，重新修改之前相关的API；zhanglogging-prefect.jar；
	5、JDBC---数据库驱动；
	写了一个统一的接口层；日志门面（日志的一个抽象层）；logging-abstract.jar；
	给项目中导入具体的日志实现就行了；我们之前的日志框架都是实现的抽象层；



### 1.1、市面上的日志框架；

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面  （日志的抽象层）                                   | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ~~JCL（Jakarta  Commons Logging（apache的））~~    SLF4j（Simple  Logging Facade for Java）    **~~jboss-logging~~** | Log4j  JUL（java.util.logging）  Log4j2  **Logback** |

左边选一个门面（抽象层）、右边来选一个实现；

日志门面：  SLF4J；

日志实现：Logback；



注意：**slf4j跟log4j、logback是一个人写的，适配性要好。logback是log4j的一个加强版。**

**log4j2是Apache的，是借着log4j的名字，性能很好。但市面上对其适配性不是很好，所以大部分用的是slf4j。**





**SpringBoot：底层是Spring框架，Spring框架默认是用JCL；common-logging**

**==SpringBoot选用 SLF4j和logback；==**



## 2、SLF4j（抽象层）使用

> ==以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，**而是调用日志抽象层里面的方法**==,然后通过抽象层中的方法调用具体实现层里面的方法。

### 2.1、如何在系统中使用SLF4j   https://www.slf4j.org

==以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，**而是调用日志抽象层里面的方法**==

给系统里面导入slf4j的jar和  logback的实现jar

导包使用slf4j的包

```java
import org.slf4j.Logger; 
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

图示；

![concrete-bindings](.\assets\concrete-bindings-1561960269842.png)

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成日志实现框架自己本身的配置文件；**



**注意：log4j出现的比较早，没有想到slf4j这回事，所以当slf4j作为门面（统一抽象层，调用的还是调用统一抽象层，里面具体的实现就是log4j或者logback）时，log4j跟slf4j需要一个适配层——slf4j-log412.jar，这个适配包，上面实现了slf4j的方法，方法里面要实现真正的日志记录就调用了log4j的具体方法。**





### 2.2、遗留问题

a系统：（slf4j+logback）: Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、xxxx

统一日志记录，即使是别的框架和我一起统一使用slf4j进行输出？

**注意:spring底层用的是commons-logging日志框架。Hibernate底层用的是jboss-logging日志框架。mybatis底层用的是slf4j+slf4j-log412.jar+log4j作为日志框架。**

![legacy](.\assets\legacy.png)

**如何让系统中所有的日志都统一到slf4j；**

==1、将系统中其他日志框架先排除出去；==

==2、用中间包来替换原有的日志框架；==

==3、我们导入slf4j其他的实现==



**总结：**因为一开始我们使用日志的时候，用的是slf4j的门面，那么有一些框架底层用的不是slf4j日志框架，所以要把这些框架底层的日志框架替换成支持slf4j框架的（其实就是替换原来的jar包，使现在的jar包里面方法跟原来jar包的方法一样，但是替换的jar包里面调用的是slf4j里面的方法，相当于在slf4j上面又加了一层适配包），然后就对应一些jar包开发了一些对应slf4j支持的适配包，同样支持原来的框架。





**中间包**：其实就是替换原来的jar包（例如common-logging、jboss-logging），使现在的jar包（jul-to-slf4j、jcl-over-slf4j）里面方法跟原来jar包（common-logging、jboss-logging）的方法一样，但是替换的jar包（jul-to-slf4j、jcl-over-slf4j））里面调用的是slf4j里面的方法，相当于在slf4j上面又加了一层适配包；

原来包的方法名跟包类的名字并没有变，只是修改了方法的实现，调用了slf4j的的方法。然后slf4j再调用具体的实现。



```java
private  Logger logger = LoggerFactory.getLogger(HelloController.class);
```



## 3、SpringBoot日志关系

### 3.1、SpringBoot使用它来做日志功能；

```xml
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
	</dependency>
```

### 3.2、底层依赖关系

1.5.10版本

![搜狗截图20180131220946](.\assets\搜狗截图20180131220946.png)



2.1.6版本

![1562454911830](.\assets\1562454911830.png)







### 3.3、总结：

#### 1）、SpringBoot底层也是使用slf4j+logback的方式进行日志记录
#### 2）、SpringBoot也把其他的日志都替换成了slf4j；
#### 3）、中间替换包？



```java
@SuppressWarnings("rawtypes")
public abstract class LogFactory {

    static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";

    static LogFactory logFactory = new SLF4JLogFactory();
}
```



![搜狗截图20180131221411](.\assets\搜狗截图20180131221411.png)

#### 4）、如果我们要引入其他框架？一定要把这个框架的默认日志依赖移除掉？
Spring框架用的是commons-logging**；所以要排除这个依赖。**



```xml
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<exclusions>
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```

**==SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可；==**



## 4、日志使用；

### 4.1、默认配置

SpringBoot默认帮我们配置好了日志；

```java
	//记录器
	Logger logger = LoggerFactory.getLogger(getClass());
	@Test
	public void contextLoads() {
		//System.out.println();

		//日志的级别；
		//由低到高   trace<debug<info<warn<error
		//可以调整输出的日志级别；日志就只会在这个级别及以后的高级别生效
		logger.trace("这是trace日志...");
		logger.debug("这是debug日志...");
		//SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别；root级别
		logger.info("这是info日志...");
		logger.warn("这是warn日志...");
		logger.error("这是error日志...");


	}
```

#### 4.1.1、**总结：**

##### 1）、创建日志记录器

```
Logger logger = LoggerFactory.getLogger(getClass());
```

##### 2）、日志级别

（1）、由低到高   trace<debug<info<warn<error
（2）、可以调整输出的日志级别；日志就只会在这个级别及以后的高级别生效
（3）、SpringBoot默认给我们使用的是info级别的（所以只会输出info级别以及info级别以后的：info<warn<error），没有指定级别的就用SpringBoot默认规定的级别；root级别



        日志输出格式：
    		%d表示日期时间，
    		%thread表示线程名，
    		%-5level：级别从左显示5个字符宽度
    		%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
    		%msg：日志消息，
    		%n是换行符
        -->
        %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n


#### 4.4.2、SpringBoot修改日志的默认配置application.properties

```properties
logging.level.com.nieshenkuan=trace

#logging.path=
# 不指定路径在当前项目下生成springboot.log日志
# 可以指定完整的路径；
#logging.file=G:/springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

#  在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

| logging.file | logging.path | Example  | Description                        |
| ------------ | ------------ | -------- | ---------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                     |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件               |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

### 4.2、指定配置

>  给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了；

**logback-spring.xml**：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>

```

如：

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
```



如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误

```
 no applicable action for [springProfile]
```





## 5、切换日志框架

可以按照slf4j的日志适配图，进行相关的切换；

####  5.1、**slf4j+log4j的方式；**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>

```

![1562466356912](.\assets\1562466356912.png)



####  5.2、**切换为log4j2**

```xml
   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```



![1562466525964](.\assets\1562466525964.png)

-----------------