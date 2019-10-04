# 二、配置文件

## 1、配置文件

**SpringBoot使用一个全局的配置文件，配置文件名是固定的；**

**•application.properties**

**•application.yml**

==注意：如果application.properties中跟application.yml中都配置了端口号的话，最终生效的是properties中的端口号，意思就是先加载了application.yml文件（优先级高），然后加载application.properties文件（优先级较yml文件低）。==

配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层都给我们自动配置好；



YAML（YAML Ain't Markup Language）
```
YAML  A Markup Language：是一个标记语言
YAML  isn't Markup Language：不是一个标记语言；
```




标记语言：
以前的配置文件；大多都使用的是  **xxxx.xml**文件；
YAML：**以数据为中心**，比json、xml等更适合做配置文件；

**YAML：配置例子**

```yaml
server:
  port: 8081
```

**XML：**

```xml
<server>
	<port>8081</port>
</server>
```



## 2、YAML语法：

### 1、基本语法

k:(空格)v：表示一对键值对（空格必须有）；

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

```yaml
server:
    port: 8081
    path: /hello
```

属性和值也是大小写敏感；





### 2、值的写法

#### 2.1、字面量：普通的值（数字，字符串，布尔）
```
k: v：字面直接来写；
字符串默认不用加上单引号或者双引号；

""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
name:   "zhangsan \n lisi"：输出；zhangsan 换行  lisi
	
''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
name:   ‘zhangsan \n lisi’：输出；zhangsan \n  lisi
```





#### 2.2、对象、Map（属性和值）（键值对）：
k: v     ：在下一行来写对象的属性和值的关系；注意缩进;
对象还是k: v的方式

```yaml
friends:
		lastName: zhangsan
		age: 20
```

**行内写法：**

```yaml
friends: {lastName: zhangsan,age: 18}
```



#### 2.3、数组（List、Set）：
用- 值   表示数组中的一个元素
```yaml
pets:
 - cat
 - dog
 - pig
```

**行内写法**

```yaml
pets: [cat,dog,pig]
```



#### 以上三种示例：

```yml
person:
  lastName: hello
  age: 18
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
    - lisi
    - zhaoliu
  dog:
    name: 小狗
    age: 12
```



## 3、配置文件值注入



**配置文件**

```yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```



**javaBean：**

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```



**总结**

**总结**：将配置文件中配置的每一个属性的值，映射到这个组件中，使用`@ConfigurationProperties`：

**`@ConfigurationProperties`注解**

> 告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；prefix = "person"：配置文件中哪个下面的所有属性进行一一映射，同时这个组件要放在容器中，所以：只有这个组件是容器中的组件，才能容器提供的;
>
> @ConfigurationProperties功能；要在组件上加@Component。

==**@ConfigurationProperties 跟 @Component注解一起使用：**==





**pom文件中导入配置文件处理器**

我们可以在pom文件中导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-configuration-processor</artifactId>
<optional>true</optional>
</dependency>
```



#### 1、properties配置文件在idea中默认utf-8可能会乱码

**解决方案**

![1560995564526](.\assets\1560995564526.png)







#### 2、@Value获取值和@ConfigurationProperties获取值比较（重要）

|                                     | @ConfigurationProperties | @Value                  |
| ----------------------------------- | ------------------------ | ----------------------- |
| 功能                                | 批量注入配置文件中的属性 | 一个个指定              |
| 松散绑定（松散语法）                | 支持                     | 不支持                  |
| SpEL                                | 不支持                   | 支持                    |
| JSR303数据校验                      | 支持                     | 不支持                  |
| 复杂类型封装（Map，List，对象等等） | 支持                     | 不支持（map、list等等） |



**配置文件yml还是properties他们都能获取到值；**

如果说，**我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用`@Value`**；`@Value`注解跟`@ConfigurationProperties`注解是分开用的。不要混淆。

如果说，我们**专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；**





#### 3、配置文件注入值数据校验`@Component`+`@ConfigurationProperties`

**支持数据校验的是`@ConfigurationProperties`注解，`@Value`是不支持的。**

这里以邮箱校验为例，Javabean为下面这个。注意，一定要加`@ConfigurationProperties`注解，一开始我没有加，导致校验失效了，无论写什么，都可以通过。**这里`@Component`跟`@ConfigurationProperties`一定要配合使用。**

```java
@Component
@ConfigurationProperties(prefix = "test")
@Validated
public class TestValueAndValidated {

    @Email
    //这个属性一定要是一个邮箱格式的，如果不是邮箱格式的。则校验失败
    @Value("nieshenkuan")
    private String name;

    @Value("10")
    private String age;


    @Value("${random.value}")
    private String strValue;

    @Value("${random.int[1024,65536]}")
    private Integer randomNum;


    @Value("${random.int}")
    private Integer randomNum1;

    @Value("${random.long}")
    private Long randomNum2;

    @Value("${random.int(10)}")
    private Integer randomNum3;
	
    //set、get方法
	//toString方法
}

```

yml文件如下：

```ymal
test:
  name: dd
```

运行SpringBoot的测试类，出现这样：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootHellloworldApplicationTests {
    @Autowired
    TestValueAndValidated testValueAndValidated;
    @Test
    public void contextLoads() {
        System.out.println(testValueAndValidated);
    }

}

```

结果如下：

![1561858415389](.\assets\1561858415389.png)







#### 4、`@PropertySource`&`@ImportResource`&`@Bean`

##### **4.1、@PropertySource：加载指定的配置文件；**

> @PropertySource：加载指定的配置文件
>
> @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
>
> prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
>
> 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
>
> **@ConfigurationProperties(prefix = "person3")默认从全局配置文件中获取值；但是要是指定了配置文件的位置，就从指定的配置文件中获取值**

**如果不加这个注解，就从默认的application.properties以及application.yml文件中去找对应的属性。但是加了这个注解，就从指定的文件中去找指定的属性的值。**

**Person3.java（javabean）**

```java
/**
 * @Description: @PropertySource：加载指定的配置文件；学习
 * @PropertySource：加载指定的配置文件
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 * prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *  @ConfigurationProperties(prefix = "person3")默认从全局配置文件中获取值；但是要是指定了配置文件的位置，就从指定的配置文件中获取值
 * @author: nieshenkuan
 * @Date: 2019/6/20 20:56
 */
@PropertySource(value = {"classpath:person3.properties"})
@Component
@ConfigurationProperties(prefix = "person3")
public class Person3 {

    private String lastName;
    private Integer age;
    private Integer intValue1;
    private Integer intValue2;
    private Integer intValue3;
    private Long longValue;
    private String  strValue;
    private String strUUID;
    //set、get方法
    //toString方法
}
```

**person3.properties**

```properties
person3.lastName=nieshenkuan
person3.age=18
person3.intValue1=${random.int}
person3.intValue2=${random.int(10)}
person3.intValue3=${random.int[1024,65536]}
person3.longValue=${random.long}
person3.strValue=${random.value}
person3.strUUID=${random.uuid}
```









##### **4.2、`@ImportResource`：导入Spring的配置文件，让配置文件里面的内容生效；**

> `@ImportResource`：导入Spring的配置文件，让配置文件里面的内容生效；
>
> 这个注解一般是加在配置类上。一般是SpringBoot主启动类上。看下面的例子：
>
> **但是Springboot推荐使用全注解的方式，不再写这种xml的配置文件。所以我们要替换xml的这种方式。**，推荐使用**@Configuration跟@Bean两个注解组合使用**，给容器中添加bean。



Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来；**`@ImportResource`**标注在一个配置类上（==Springboot主配置类上==）

```java
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class SpringBoot01HelloworldQuickApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01HelloworldQuickApplication.class, args);
    }
}
导入Spring的配置文件让其生效
```



Spring的配置文件bean.xml以及Java类。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="helloService" class="com.nieshenkuan.springboot.service.HelloService"></bean>
</beans>
```

```java
public class HelloService {
}
```



测试：

```java
/**
 *可以在测试期间很方便的类似编码一样进行自动注入等容器的功能。
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootHellloworldApplicationTests {
    @Autowired
    ApplicationContext ioc;

    @Test
    public void test01(){
        boolean b = ioc.containsBean("helloService");
        System.out.println(b);
    }
}
```



返回true

**注意：**

**但是Springboot推荐使用全注解的方式，不再写这种xml的配置文件。所以我们要使用下面这种方式。**

**@Configuration跟@Bean组合使用**

下面来看一下**@Configuration跟@Bean组合使用**：



##### **4.3、`@Bean` +`@Configuration`**

==重点：==

>  SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式
>
> **@Configuration跟@Bean组合使用**
>
> 1、配置类**@Configuration**----相当于---->Spring的配置文件
>
> 2、使用**@Bean**给容器中添加组件，需要new一个对象，或者通过其他方式给容器添加一个对象。





1、配置类**@Configuration**----相当于---->Spring配置文件

2、使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

HelloService.java

```java
public class HelloService {
    private String name;
    private Integer age;
}
```



测试：

```java
/**
 *可以在测试期间很方便的类似编码一样进行自动注入等容器的功能。
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootHellloworldApplicationTests {

  
    @Autowired
    ImportResourceService importResourceService;

    @Autowired
    ApplicationContext ioc;

    @Test
    public void test01(){
        boolean b = ioc.containsBean("importResourceService");
        boolean c = ioc.containsBean("helloService");
        System.out.println(b); //true
        System.out.println(c); //true
    }
}
```



## 4、配置文件占位符

### 1、随机数

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}

```



### 2、占位符获取之前配置的值，如果没有可以是用:指定默认值

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```



**示例：**

`${person.hello:hello}_dog`:要是没有person.hello这个值的话，就用：后面的值为默认值



## 5、Profile

> **Profile是Spring对不同环境提供不同配置功能的支持，可以通过激活、指定参数等方式快速切换环境**;
>
> 我们在主配置文件编写的时候，文件名可以是   `application-{profile}.properties/yml`
>
> 默认使用application.properties的配置；
>
> **也可以在application.properties中指定使用哪个profile文件**：`spring.profiles.active=dev`;
>
> yml支持多文档块方式（文档块的形式）
>
>
>
>



### 1、多Profile文件

**Profile是Spring对不同环境提供不同配置功能的支持，可以通过激活、指定参数等方式快速切换环境**

我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties/yml

默认使用application.properties的配置；

**也可以在application.properties中指定使用哪个profile文件**：

```
spring.profiles.active=dev
```

### 2、yml支持多文档块方式（文档块的形式）

```yml

server:
  port: 8081
spring:
  profiles:
    active: prod  #指定使用哪个环境

---
server:
  port: 8083
spring:
  profiles: dev


---

server:
  port: 8084
spring:
  profiles: prod 
```

在yaml中，---代表一个文档块



### 3、激活指定profile的方式

#### 3.1、在配置文件中指定  --spring.profiles.active=dev

![1561863644574](.\assets\1561863644574.png)





#### 3.2、命令行：

```java
java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar      --spring.profiles.active=dev；
```

可以直接在测试的时候，即启动jar的时候（通过java命令启动项目），配置传入命令行参数

#### 3.3、虚拟机参数；

```java
-Dspring.profiles.active=dev
```



![1561863972255](.\assets\1561863972255.png)







## 6、配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

| 路径                | 含义                                                   |
| ------------------- | ------------------------------------------------------ |
| –file:./config/     | （当前项目的config文件夹下的默认配置文件）：优先级最高 |
| –file:./            | （当前项目下的默认配置文件）：次之                     |
| –classpath:/config/ | （类路径下的config文件夹下面的默认配置文件）           |
| –classpath:/        | （类路径下的默认配置文件）                             |
|                     |                                                        |

以上优先级由高到底，高优先级的配置会覆盖低优先级的配置；（实践也是如此）

![1561969398952](.\assets\1561969398952.png)

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；但是，像这种maven项目，打包的时候不会将项目下的config目录下的配置文件以及项目目录下的application.properties文件给打包进去，**所以，一般会在jar包外放一个application.properties文件，这样的话，可以通过命令行的形式来指定配置文件的位置。实践中也是如此（宿迁热网开发就是这样）通过spring.config.location来改变默认的配置文件位置**



**==我们还可以通过spring.config.location来改变默认的配置文件位置==**

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

运行jar的时候可以指定外部的配置文件，那么我们就不用该原项目中的配置，只是直接用外部的配置文件：如下：

```
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties
```

**通过spring.config.location属性指定外部的配置文件的位置（很重要）。**



## 7、外部配置加载顺序

**==SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置==**

### 7.1 **命令行参数**（比较常用）

所有的配置都可以在命令行上进行指定

```java
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
```

多个配置用空格分开； --配置项=值



### 7.2.来自java:comp/env的JNDI属性



### 7.3.Java系统属性（System.getProperties()）



### 7.4.操作系统环境变量



### 7.5.RandomValuePropertySource配置的random.*属性值

==**由jar包外向jar包内进行寻找；**==

==**优先加载带profile**==



###  7.**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**



### 7.**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

==**再来加载不带profile**==



### 7.**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**



### 7.**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**



### 7.10.@Configuration注解类上的@PropertySource

### 7.11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源；

[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)



## 8、自动配置原理(boot的核心)

==springboot的精华==

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties)

![1561959674711](.\assets\1561959674711.png)

```
Spring Boot 支持多种外部配置方式
这些方式优先级如下：
https://docs.spring.io/spring-boot/docs/currentSNAPSHOT/reference/htmlsingle/#boot-features-external-config
1. 命令行参数
2. 来自java:comp/env的JNDI属性
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. RandomValuePropertySource配置的random.*属性值
6. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
7. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件
8. jar包外部的application.properties或application.yml(不带spring.profile)配置文件
9. jar包内部的application.properties或application.yml(不带spring.profile)配置文件
10. @Configuration注解类上的@PropertySource
11. 通过SpringApplication.setDefaultProperties指定的默认属性
```



### 1、**自动配置原理：**

#### 1）、SpringBoot启动的时候加载主配置类，开启了自动配置功能 ==@EnableAutoConfiguration(核心注解)==

#### **2）、@EnableAutoConfiguration 作用：**

 - 利用EnableAutoConfigurationImportSelector给容器中导入一些组件？

- 可以查看selectImports()方法的内容；

- List<String> configurations = getCandidateConfigurations(annotationMetadata,      attributes);获取候选的配置
-  SpringFactoriesLoader.loadFactoryNames()扫描所有jar包类路径下  META-INF/spring.factories把扫描到的这些文件的内容包装成properties对象； 从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中   


**==将 类路径下  META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中；==**

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
```

每一个这样的  xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；



#### 3）、每一个自动配置类进行自动配置功能；



#### 4）、以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；

```java
@Configuration   //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中（给加到容器中去，这点很重要，因为HttpEncodingProperties类上面只有一个@ConfigurationProperties注解，并没有@Componnet注解，就是将这个类加到容器中取，当前这个注解就将@ConfigurationProperties注解的HttpEncodingProperties类加入到容器中去了。）

@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；    判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class)  //判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {
  
  	//他已经和SpringBoot的配置文件映射了
  	private final HttpEncodingProperties properties;
  
   //只有一个有参构造器的情况下，参数的值就会从容器中拿
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

根据当前不同的条件判断，决定这个配置类是否生效？

一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；



`@EnableConfigurationProperties(HttpEncodingProperties.class) ` ：**启动指定类的ConfigurationProperties功能**；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中（给加到容器中去，这点很重要，因为HttpEncodingProperties类上面只有一个@ConfigurationProperties注解，并没有@Componnet注解，就是将这个类加到容器中取，当前这个注解就将@ConfigurationProperties注解的HttpEncodingProperties类加入到容器中去了。）

详细看一下`@EnableConfigurationProperties(HttpEncodingProperties.class)`注解

**@EnableConfigurationProperties注解**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({EnableConfigurationPropertiesImportSelector.class})
public @interface EnableConfigurationProperties {
    Class<?>[] value() default {};
}
```



**EnableConfigurationPropertiesImportSelector.java类**

```java
class EnableConfigurationPropertiesImportSelector implements ImportSelector {
    EnableConfigurationPropertiesImportSelector() {
    }

    public String[] selectImports(AnnotationMetadata metadata) {
        MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(EnableConfigurationProperties.class.getName(), false);
        Object[] type = attributes == null ? null : (Object[])((Object[])attributes.getFirst("value"));
        return type != null && type.length != 0 ? new String[]{EnableConfigurationPropertiesImportSelector.ConfigurationPropertiesBeanRegistrar.class.getName(), ConfigurationPropertiesBindingPostProcessorRegistrar.class.getName()} : new String[]{ConfigurationPropertiesBindingPostProcessorRegistrar.class.getName()};
    }

    public static class ConfigurationPropertiesBeanRegistrar implements ImportBeanDefinitionRegistrar {
        public ConfigurationPropertiesBeanRegistrar() {
        }

        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(EnableConfigurationProperties.class.getName(), false);
            List<Class<?>> types = this.collectClasses((List)attributes.get("value"));
            Iterator var5 = types.iterator();

            while(var5.hasNext()) {
                Class<?> type = (Class)var5.next();
                String prefix = this.extractPrefix(type);
                String name = StringUtils.hasText(prefix) ? prefix + "-" + type.getName() : type.getName();
                if (!registry.containsBeanDefinition(name)) {
                    this.registerBeanDefinition(registry, type, name);
                }
            }

        }

        private String extractPrefix(Class<?> type) {
            ConfigurationProperties annotation = (ConfigurationProperties)AnnotationUtils.findAnnotation(type, ConfigurationProperties.class);
            return annotation != null ? annotation.prefix() : "";
        }

        private List<Class<?>> collectClasses(List<Object> list) {
            ArrayList<Class<?>> result = new ArrayList();
            Iterator var3 = list.iterator();

            while(var3.hasNext()) {
                Object object = var3.next();
                Object[] var5 = (Object[])((Object[])object);
                int var6 = var5.length;

                for(int var7 = 0; var7 < var6; ++var7) {
                    Object value = var5[var7];
                    if (value instanceof Class && value != Void.TYPE) {
                        result.add((Class)value);
                    }
                }
            }

            return result;
        }

        private void registerBeanDefinition(BeanDefinitionRegistry registry, Class<?> type, String name) {
            BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(type);
            AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
            registry.registerBeanDefinition(name, beanDefinition);
            ConfigurationProperties properties = (ConfigurationProperties)AnnotationUtils.findAnnotation(type, ConfigurationProperties.class);
            Assert.notNull(properties, "No " + ConfigurationProperties.class.getSimpleName() + " annotation found on  '" + type.getName() + "'.");
        }
    }
}

```



#### 自动配置原理示例步骤思路总结

> 将 类路径下  META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中（这里只是以其中一个为例）

![1568903264337](.\assets\1568903264337.png)

![1568904874572](.\assets\1568904874572.png)

#####     (1)、找到META-INF/spring.factories文件

![1568903369975](.\assets\1568903369975.png)

##### （2）、找到示例的HttpEncodingAutoConfiguration类，点击进去

`@Configuration`:这是一个配置类，相当于spring的xml文件

`@EnableConfigurationProperties`:**启动指定类的ConfigurationProperties功能**；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中《HttpEncodingProperties这个类上面并没有加@Component注解》（给加到容器中去，这点很重要，因为HttpEncodingProperties类上面只有一个@ConfigurationProperties注解，并没有@Componnet注解，就是将这个类加到容器中取，当前这个注解就将@ConfigurationProperties注解的HttpEncodingProperties类加入到容器中去了。）

![1568903393056](.\assets\1568903393056.png)

##### （3）、点击进@EnableConfigurationProperties注解中HttpProperties类

![1568903406148](.\assets\1568903406148.png)

这就很熟悉了。这个类用@ConfigurationProperties注解标注了，说明跟配置文件中某前缀标志的属性像绑定了。

@ConfigurationProperties(prefix = "spring.http")

所有配置文件中以spring.http开头的都将跟HttpProperties类中的属性相绑定。

通过上面几个步骤，就可以到我们一开始的类与配置文件相绑定





#### 5）、所有在配置文件中能配置的属性都是在xxxxProperties（HttpProperties）类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
    
}
```



springboot的版本不同，上面导入的properties文件也不同。下面那两幅图就是：

第一幅是1.5.9版本，这里导入的HttpEncodingAutoConfiguration类上面自动配置的是HttpEncodingProperties。

![1561002283801](.\assets\1561002283801.png)





第二幅是2.1.6版本的springboot，这里导入的HttpEncodingAutoConfiguration类上面自动配置的是HttpProperties。这里用了Http的总的properties的文件。

![1561002351075](.\assets\1561002351075.png)

**精髓：**

**1）、SpringBoot启动会加载大量的自动配置类**

**2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；**

**3）、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）**

**4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**



xxxxAutoConfigurartion：自动配置类；

给容器中添加组件

xxxxProperties:封装配置文件中相关属性；

这两个文件一一对应，然后我们就可以在配置文件中定义自动配置类中的值。





### 2、细节

#### 1、@Conditional派生注解（Spring注解版原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效；**

我们怎么知道哪些自动配置类生效；

**==我们可以通过启用  debug=true属性；来让控制台打印自动配置报告==**，这样我们就可以很方便的知道哪些自动配置类生效；

```java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
        
```