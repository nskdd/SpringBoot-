# 十二、Spring Boot与任务

（异步任务、定时任务、邮件任务）

## 一、异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了**@Async(注解）**来完美解决这个问题。

核心的两个注解：
**（1）、@EnableAysnc：用在springboot的主应用上，开启异步注解**

**（2）、@Aysnc：用在方法上，说名该方法时一个异步方法。**

示例：

异步方法：

```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

/**
 * @Description: 异步任务类
 * @author: nieshenkuan
 * @Date: 2019/8/1 20:36
 */
@Service
public class AsyncService {
    //告诉Spring这是一个异步方法
    @Async
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中...");
    }
}

```

请求异步方法的controller层：

```java
/**
 * @Description: 异步任务访问的controller层
 * @author: nieshenkuan
 * @Date: 2019/8/1 20:37
 */
@RestController
public class AsyncController {

    @Autowired
    AsyncService asyncService;

    @GetMapping("/hello")
    public String hello(){
        asyncService.hello();
        return "success";
    }
}

```



然后需要在boot的主应用类上开启异步注解：

```java
@SpringBootApplication
@EnableAsync//开启异步注解功能
public class SpringbootTaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTaskApplication.class, args);
    }

}

```



## 二、定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，**提供TaskExecutor 、TaskScheduler 接口。**

两个注解：

**(1)@EnableScheduling:用在boot的主应用上：开启基于注解的定时任务**

**(2)@Scheduled：用在方法上。说明这个方法是一个定时方法**

其中，定时方法上的注解里面的表达式核心如下：

cron表达式：

| 字段 | 允许值                | 允许的特殊字符  |
| ---- | --------------------- | --------------- |
| 秒   | 0-59                  | , - * /         |
| 分   | 0-59                  | , - * /         |
| 小时 | 0-23                  | , - * /         |
| 日期 | 1-31                  | , - * ? / L W C |
| 月份 | 1-12                  | , - * /         |
| 星期 | 0-7或SUN-SAT 0,7是SUN | , - * ? / L C # |



| 特殊字符 | 代表含义                   |
| -------- | -------------------------- |
| ,        | 枚举                       |
| -        | 区间                       |
| *        | 任意                       |
| /        | 步长                       |
| ?        | 日/星期冲突匹配            |
| L        | 最后                       |
| W        | 工作日                     |
| C        | 和calendar联系后计算过的值 |
| #        | 星期，4#2，第2个星期四     |



定义一个定时任务：

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

/**
 * @Description: boot中的定时任务
 * @author: nieshenkuan
 * @Date: 2019/8/1 21:19
 */
@Service
public class ScheduledService {

    /**
     * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）.
     * 0 * * * * MON-FRI
     *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
     *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
     *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
     *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
     *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
     */
    // @Scheduled(cron = "0 * * * * MON-SAT")
    //@Scheduled(cron = "0,1,2,3,4 * * * * MON-SAT")
    // @Scheduled(cron = "0-4 * * * * MON-SAT")
    @Scheduled(cron = "0/4 * * * * MON-SAT")  //每4秒执行一次
    public void hello(){
        System.out.println("定时任务被调用了... ... ");
    }
}

```



在boot主应用上开启基于注解的定时任务

```java
@EnableAsync  //开启异步注解功能
@EnableScheduling //开启基于注解的定时任务
@SpringBootApplication
public class Springboot04TaskApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot04TaskApplication.class, args);
	}
}

```





## 三、邮件任务

邮件发送需要引入spring-boot-starter-mail
Spring Boot 自动配置MailSenderAutoConfiguration
定义MailProperties内容，配置在application.yml中
自动装配JavaMailSender
测试邮件发送

![1561963488472](.\assets\1561963488472.png)

#### pom.xml

```xml
<dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-mail</artifactId>
 </dependency>
```

#### application.properties

```properties
spring.mail.username=853099781@qq.com
spring.mail.password=xvfjircgwsatbcha
spring.mail.host=smtp.qq.com
spring.mail.properties.mail.smtp.ssl.enable=true
```

#### 发送邮件

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootTaskApplicationTests {

    @Autowired
    JavaMailSenderImpl mailSender;

    @Test
    public void contextLoads() {
        SimpleMailMessage message = new SimpleMailMessage();
        //邮件设置
        message.setSubject("测试（信头）");
        message.setText("这是信内容");

        message.setTo("1208221986@qq.com");
        message.setFrom("853099781@qq.com");

        mailSender.send(message);
    }

    @Test
    public void test02() throws  Exception{
        //1、创建一个复杂的消息邮件
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

        //邮件设置
        helper.setSubject("文件头");
        helper.setText("<b style='color:red'>文件内容，这是复杂邮件---我爱你！</b>",true);

        helper.setTo("1208221986@qq.com");
        helper.setFrom("853099781@qq.com");

        //上传文件
        helper.addAttachment("1.jpg",new File("D:\\image\\lyy.jpg"));
        helper.addAttachment("2.jpg",new File("D:\\image\\DSC_7499.JPG"));

        mailSender.send(mimeMessage);

    }

}

```





**springboot-task项目**