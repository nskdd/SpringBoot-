# 九、Spring Boot与缓存

（JSR-107、Spring缓存抽象、整合Redis）

## 一、JSR107（jsr107缓存规范）

Java Caching定义了5个核心接口，分别是**CachingProvider, CacheManager, Cache, Entry** 和 **Expiry**。

* **CachingProvider**：（缓存提供者）定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
* **CacheManager**：（缓存管理器）定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
* **Cache：**（缓存）是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
* **Entry：**是一个存储在Cache中的key-value对。
* **Expiry：** 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。



![1561961336076](.\assets\1561961336076.png)



## 二、Spring缓存抽象

Spring从3.1开始定义了org.springframework.cache.**Cache**
和org.springframework.cache.**CacheManager**接口来统一不同的缓存技术；
并支持使用JCache（JSR-107）注解简化我们开发；

* Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
* Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；
* 每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
* 使用Spring缓存抽象时我们需要关注以下两点；
1、确定方法需要被缓存以及他们的缓存策略
2、从缓存中读取之前缓存存储的数据



![1561961377621](.\assets\1561961377621.png)

## 三、几个重要概念&缓存注解

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| -------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件                        |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存     |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，又希望结果被缓存。                           |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |



@Cacheable/@CachePut/@CacheEvict 主要的参数

| @Cacheable/@CachePut/@CacheEvict 主要的参数 |
| ------------------------------------------- |
|                                             |

![1561961690734](.\assets\1561961690734.png)



Cache SpEL available metadata

| 名字          | 位置               | 描述                                                         | 示例                 |
| ------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method        | root object        | 当前被调用的方法                                             | #root.method.name    |
| target        | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass   | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args          | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches        | root object        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1", "cache2"})），则有两个cache | #root.caches[0].name |
| argument name | evaluation context | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引； | #iban 、 #a0 、  #p0 |
| result        | evaluation context | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式 ’cache evict’的表达式beforeInvocation=false） | #result              |

## 四、缓存使用

1、引入spring-boot-starter-cache模块(注意，这里不能引入其他cache的实现，不能默认的就不生效了。)
2、@EnableCaching开启缓存（在主配置类上）
3、使用缓存注解
4、切换为其他缓存

#### 4.1、环境搭建

###### （1）、新建一个springboot工程。

###### （2）、pom文件如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.nieshenkuan</groupId>
    <artifactId>springboot-cache</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-cache</name>
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
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
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

**这里暂时用的是默认的缓存。不导入redis的依赖。**



##### 4.2、具体使用

###### （1）、两个Bean

```java
package com.nieshenkuan.bean;

public class Department {
	
	private Integer id;
	private String departmentName;
	
	
	public Department() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Department(Integer id, String departmentName) {
		super();
		this.id = id;
		this.departmentName = departmentName;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getDepartmentName() {
		return departmentName;
	}
	public void setDepartmentName(String departmentName) {
		this.departmentName = departmentName;
	}
	@Override
	public String toString() {
		return "Department [id=" + id + ", departmentName=" + departmentName + "]";
	}

}

```

```java
package com.nieshenkuan.bean;

import java.io.Serializable;

public class Employee implements Serializable {
	
	private Integer id;
	private String lastName;
	private String email;
	private Integer gender; //性别 1男  0女
	private Integer dId;
	
	
	public Employee() {
		super();
	}

	
	public Employee(Integer id, String lastName, String email, Integer gender, Integer dId) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
		this.dId = dId;
	}
	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public Integer getGender() {
		return gender;
	}
	public void setGender(Integer gender) {
		this.gender = gender;
	}
	public Integer getdId() {
		return dId;
	}
	public void setdId(Integer dId) {
		this.dId = dId;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + ", dId="
				+ dId + "]";
	}
	
	

}

```

###### (2)、两个mapper

```java
package com.nieshenkuan.mapper;

import com.nieshenkuan.bean.Department;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper
public interface DepartmentMapper {

    @Select("SELECT * FROM department WHERE id = #{id}")
    Department getDeptById(Integer id);
}

```

```java
package com.nieshenkuan.mapper;

import com.nieshenkuan.bean.Employee;
import org.apache.ibatis.annotations.*;

@Mapper
public interface EmployeeMapper {

    @Select("SELECT * FROM employee WHERE id = #{id}")
    public Employee getEmpById(Integer id);

    @Update("UPDATE employee SET lastName=#{lastName},email=#{email},gender=#{gender},d_id=#{dId} WHERE id=#{id}")
    public void updateEmp(Employee employee);

    @Delete("DELETE FROM employee WHERE id=#{id}")
    public void deleteEmpById(Integer id);

    @Insert("INSERT INTO employee(lastName,email,gender,d_id) VALUES(#{lastName},#{email},#{gender},#{dId})")
    public void insertEmployee(Employee employee);

    @Select("SELECT * FROM employee WHERE lastName = #{lastName}")
    Employee getEmpByLastName(String lastName);
}

```

###### (3)、两个service

```java
package com.nieshenkuan.service;

import com.nieshenkuan.bean.Department;
import com.nieshenkuan.mapper.DepartmentMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.cache.Cache;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.stereotype.Service;


@Service
public class DeptService {

    @Autowired
    DepartmentMapper departmentMapper;

    @Qualifier("deptCacheManager")
    @Autowired
    RedisCacheManager deptCacheManager;


    /**
     *  缓存的数据能存入redis；
     *  第二次从缓存中查询就不能反序列化回来；
     *  存的是dept的json数据;CacheManager默认使用RedisTemplate<Object, Employee>操作Redis
     *
     *
     * @param id
     * @return
     */
//    @Cacheable(cacheNames = "dept",cacheManager = "deptCacheManager")
//    public Department getDeptById(Integer id){
//        System.out.println("查询部门"+id);
//        Department department = departmentMapper.getDeptById(id);
//        return department;
//    }

    // 使用缓存管理器得到缓存，进行api调用
    public Department getDeptById(Integer id){
        System.out.println("查询部门"+id);
        Department department = departmentMapper.getDeptById(id);

        //获取某个缓存
        Cache dept = deptCacheManager.getCache("dept");
        dept.put("dept:1",department);

        return department;
    }


}

```



```java
package com.nieshenkuan.service;

import com.nieshenkuan.bean.Employee;
import com.nieshenkuan.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;
/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/7/28 13:32
 */
@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
@Service
public class EmployeeService {

    @Autowired
    EmployeeMapper employeeMapper;

    /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
     * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
     *

     *
     * 原理：
     *   1、自动配置类；CacheAutoConfiguration
     *   2、缓存的配置类
     *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *   3、哪个配置类默认生效：SimpleCacheConfiguration；
     *
     *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager（可以点进SimpleCacheConfiguration查看）
     *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
     *
     *   运行流程：
     *   @Cacheable：运行例子；
     *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
     *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
     *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
     *          SimpleKeyGenerator生成key的默认策略；
     *                  如果没有参数；key=new SimpleKey()；
     *                  如果有一个参数：key=参数的值
     *                  如果有多个参数：key=new SimpleKey(params)；
     *   3、没有查到缓存就调用目标方法；
     *   4、将目标方法返回的结果，放进缓存中
     *
     *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     *
     *   核心：
     *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     *
     *   几个属性：
     *      cacheNames/value：指定缓存组件的名字(可以理解时一组缓存集合的名字);将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
     *              编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
     *              getEmp[2]
     *
     *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
     *              key/keyGenerator：二选一使用;
     *
     *
     *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *              ,condition = "#id>0"
     *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
     *
     *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null"
     *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
     *      sync：是否使用异步模式:默认为false
     *      如果为true，即不支持unless属性
     * @param id
     * @return
     *
     * @Cacheable(value = {"emp"}) ==  @Cacheable(cacheNames = {"emp"})
     *  * 运行时机：
     *      *  1、在调用目标方法前，先到缓存中查找是否有缓存对应的值，如果有就去缓存中找，不执行目标方法。如果没有，执行2.
     *      *  2、执行目标方法，并执行3
     *         3、将查询的数据缓存起来。
     */
    //value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/
    @Cacheable(value = {"emp"})
    public Employee getEmployeeById(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee empById = employeeMapper.getEmpById(id);
        return empById;
    }

    /**
     * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时更新缓存；
     * 运行时机：
     *  1、先调用目标方法
     *  2、将目标方法的结果缓存起来
     *
     * 测试步骤：
     *  1、查询1号员工；查到的结果会放在缓存中；
     *          key：1  value：lastName：张三
     *  2、以后查询还是之前的结果
     *  3、更新1号员工；【lastName:zhangsan；gender:0】
     *          将方法的返回值也放进缓存了；
     *          key：传入的employee对象  值：返回的employee对象；
     *  4、查询1号员工？
     *      应该是更新后的员工；
     *          key = "#employee.id":使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *             @Cacheable的key是不能用#result
     *      为什么是没更新前的？【1号员工没有在缓存中更新】
     *
     */
    //,key = "#result.id"
    @CachePut(value = "emp",key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *
     *  allEntries = true：指定清除这个缓存中所有的数据(一般不使用，全部清除）
     *
     *  beforeInvocation = false：缓存的清除是否在方法之前执行（默认）
     *      默认代表缓存清除操作是在方法（目标方法）执行之后执行;如果出现异常缓存就不会清除
     *
     *  beforeInvocation = true：
     *      代表清除缓存操作是在方法（目标方法）运行之前执行，无论方法是否出现异常，缓存都清除
     *
     */
    //@CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    @CacheEvict(value="emp",key = "#id",beforeInvocation = true)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        //出异常
        int i = 10/0;
    }

    // @Caching 定义复杂的缓存规则
    //@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置，类上面
    @Caching(
            cacheable = {
                    @Cacheable(/*value="emp",*/key = "#lastName")
            },
            put = {
                    @CachePut(/*value="emp",*/key = "#result.id"),
                    @CachePut(/*value="emp",*/key = "#result.email")
            }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }

}

```



###### (4)、两个controller

```java
package com.nieshenkuan.controller;

import com.nieshenkuan.bean.Department;
import com.nieshenkuan.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DeptController {

    @Autowired
    DeptService deptService;

    @GetMapping("/dept/{id}")
    public Department getDept(@PathVariable("id") Integer id){
        return deptService.getDeptById(id);
    }
}

```



```java
package com.nieshenkuan.controller;

import com.nieshenkuan.bean.Employee;
import com.nieshenkuan.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/7/28 13:34
 */
@RestController
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    @GetMapping(value = "/emp/{id}")
    public Employee getEmp(@PathVariable("id") Integer id){
        Employee employeeById = employeeService.getEmployeeById(id);
        return  employeeById;
    }

    @GetMapping("/emp")
    public Employee update(Employee employee){
        Employee emp = employeeService.updateEmp(employee);

        return emp;
    }

    @GetMapping(value="/delEmp")
    public String delEmp(Integer id){
        employeeService.deleteEmp(id);
        return "success";
    }

    @GetMapping("/emp/lastname/{lastName}")
    public Employee getEmpByLastName(@PathVariable("lastName") String lastName){
        return employeeService.getEmpByLastName(lastName);
    }
}

```

###### (5)、两个配置类

```java
package com.nieshenkuan.config;

import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * @Description: 缓存的配置类
 * @author: nieshenkuan
 * @Date: 2019/7/28 15:13
 */
@Configuration
public class MyCacheConfig {

    /**
     *@Description: 自定义缓存键的生成策略
     * key跟KeyGenerator二选一。定义了key就不用定义KeyGenerator，当然
     * 定义了KeyGenerator就不用定义key
     * keyGenerator = "myKeyGenerator"
     *@param:
     *@return:
     *@Author: nieshenkuan
     *@Date: 2019/7/28 15:15
     */
    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){

        return  new KeyGenerator() {
            @Override
            public Object generate(Object o, Method method, Object... objects) {
                return method.getName()+"["+ Arrays.asList(objects).toString()+"]";

            }
        };
    }
}

```



```java
package com.nieshenkuan.config;


import com.nieshenkuan.bean.Department;
import com.nieshenkuan.bean.Employee;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    @Bean
    public RedisTemplate<Object, Department> deptRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> ser = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(ser);
        return template;
    }



    //CacheManagerCustomizers可以来定制缓存的一些规则
    @Primary  //将某个缓存管理器作为默认的
    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }

    @Bean
    public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }


}

```

###### (6)、配置文件

```xml
spring.datasource.url=jdbc:mysql://localhost:3306/spring_cache
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

#开启驼峰命名规则
mybatis.configuration.map-underscore-to-camel-case=true

logging.level.com.nieshenkuan.mapper=debug

#debug=true

#redis地址
spring.redis.host=127.0.0.1

```

##### 4.3、核心类

```java
package com.nieshenkuan.service;

import com.nieshenkuan.bean.Employee;
import com.nieshenkuan.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;
/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/7/28 13:32
 */
@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
@Service
public class EmployeeService {

    @Autowired
    EmployeeMapper employeeMapper;

    /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
     * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
     *

     *
     * 原理：
     *   1、自动配置类；CacheAutoConfiguration
     *   2、缓存的配置类
     *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *   3、哪个配置类默认生效：SimpleCacheConfiguration；
     *
     *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager（可以点进SimpleCacheConfiguration查看）
     *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
     *
     *   运行流程：
     *   @Cacheable：运行例子；
     *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
     *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
     *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
     *          SimpleKeyGenerator生成key的默认策略；
     *                  如果没有参数；key=new SimpleKey()；
     *                  如果有一个参数：key=参数的值
     *                  如果有多个参数：key=new SimpleKey(params)；
     *   3、没有查到缓存就调用目标方法；
     *   4、将目标方法返回的结果，放进缓存中
     *
     *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     *
     *   核心：
     *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     *
     *   几个属性：
     *      cacheNames/value：指定缓存组件的名字(可以理解时一组缓存集合的名字);将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
     *              编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
     *              getEmp[2]
     *
     *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
     *              key/keyGenerator：二选一使用;
     *
     *
     *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *              ,condition = "#id>0"
     *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
     *
     *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null"
     *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
     *      sync：是否使用异步模式:默认为false
     *      如果为true，即不支持unless属性
     * @param id
     * @return
     *
     * @Cacheable(value = {"emp"}) ==  @Cacheable(cacheNames = {"emp"})
     *  * 运行时机：
     *      *  1、在调用目标方法前，先到缓存中查找是否有缓存对应的值，如果有就去缓存中找，不执行目标方法。如果没有，执行2.
     *      *  2、执行目标方法，并执行3
     *         3、将查询的数据缓存起来。
     */
    //value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/
    @Cacheable(value = {"emp"})
    public Employee getEmployeeById(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee empById = employeeMapper.getEmpById(id);
        return empById;
    }

    /**
     * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时更新缓存；
     * 运行时机：
     *  1、先调用目标方法
     *  2、将目标方法的结果缓存起来
     *
     * 测试步骤：
     *  1、查询1号员工；查到的结果会放在缓存中；
     *          key：1  value：lastName：张三
     *  2、以后查询还是之前的结果
     *  3、更新1号员工；【lastName:zhangsan；gender:0】
     *          将方法的返回值也放进缓存了；
     *          key：传入的employee对象  值：返回的employee对象；
     *  4、查询1号员工？
     *      应该是更新后的员工；
     *          key = "#employee.id":使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *             @Cacheable的key是不能用#result
     *      为什么是没更新前的？【1号员工没有在缓存中更新】
     *
     */
    //,key = "#result.id"
    @CachePut(value = "emp",key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *
     *  allEntries = true：指定清除这个缓存中所有的数据(一般不使用，全部清除）
     *
     *  beforeInvocation = false：缓存的清除是否在方法之前执行（默认）
     *      默认代表缓存清除操作是在方法（目标方法）执行之后执行;如果出现异常缓存就不会清除
     *
     *  beforeInvocation = true：
     *      代表清除缓存操作是在方法（目标方法）运行之前执行，无论方法是否出现异常，缓存都清除
     *
     */
    //@CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    @CacheEvict(value="emp",key = "#id",beforeInvocation = true)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        //出异常
        int i = 10/0;
    }

    // @Caching 定义复杂的缓存规则
    //@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置，类上面
    @Caching(
            cacheable = {
                    @Cacheable(/*value="emp",*/key = "#lastName")
            },
            put = {
                    @CachePut(/*value="emp",*/key = "#result.id"),
                    @CachePut(/*value="emp",*/key = "#result.email")
            }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }

}

```

###### 4.4、上面核心类中的方法用的注解的总结

**(1)、@CacheConfig**

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";

    String unless() default "";

    boolean sync() default false;
}

```



```java
@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
@Service
public class EmployeeService {}
```

@CacheConfig:抽取缓存的公共配置，在类上面使用。这里定义了cacheNames等一系列属性，然后在这个类中的其他方法中的一些缓存的注解也会有这个属性的值。

**（2）、@Cacheable**

```java
  /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
     * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
     *

     *
     * 原理：
     *   1、自动配置类；CacheAutoConfiguration
     *   2、缓存的配置类
     *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *   3、哪个配置类默认生效：SimpleCacheConfiguration；
     *
     *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager（可以点进SimpleCacheConfiguration查看）
     *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
     *
     *   运行流程：
     *   @Cacheable：运行例子；
     *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
     *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
     *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
     *          SimpleKeyGenerator生成key的默认策略；
     *                  如果没有参数；key=new SimpleKey()；
     *                  如果有一个参数：key=参数的值
     *                  如果有多个参数：key=new SimpleKey(params)；
     *   3、没有查到缓存就调用目标方法；
     *   4、将目标方法返回的结果，放进缓存中
     *
     *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     *
     *   核心：
     *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     *
     *   几个属性：
     *      cacheNames/value：指定缓存组件的名字(可以理解时一组缓存集合的名字);将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
     *              编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
     *              getEmp[2]
     *
     *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
     *              key/keyGenerator：二选一使用;
     *
     *
     *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *              ,condition = "#id>0"
     *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
     *
     *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null"
     *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
     *      sync：是否使用异步模式:默认为false
     *      如果为true，即不支持unless属性
     * @param id
     * @return
     *
     * @Cacheable(value = {"emp"}) ==  @Cacheable(cacheNames = {"emp"})
     *  * 运行时机：
     *      *  1、在调用目标方法前，先到缓存中查找是否有缓存对应的值，如果有就去缓存中找，不执行目标方法。如果没有，执行2.
     *      *  2、执行目标方法，并执行3
     *         3、将查询的数据缓存起来。
     */
    //value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/
    @Cacheable(value = {"emp"})
    public Employee getEmployeeById(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee empById = employeeMapper.getEmpById(id);
        return empById;
    }
```

**总结：**

将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
原理：
**1、自动配置类；CacheAutoConfiguration**

**2、缓存的配置类**  org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration  org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration  org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration  org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration  org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration **【默认】**  

org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration

**3、哪个配置类默认生效：SimpleCacheConfiguration；**

**4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager（可以点进SimpleCacheConfiguration查看）**

**5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；**
**运行流程：**
**@Cacheable：运行例子；**

1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；（CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
SimpleKeyGenerator生成key的默认策略；
如果没有参数；key=new SimpleKey()；
如果有一个参数：key=参数的值
如果有多个参数：key=new SimpleKey(params)；
3、没有查到缓存就调用目标方法；
4、将目标方法返回的结果，放进缓存中
@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
核心：
1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
几个属性：
cacheNames/value：指定缓存组件的名字(可以理解时一组缓存集合的名字);将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
getEmp[2]
keyGenerator：key的生成器；可以自己指定key的生成器的组件id
key/keyGenerator：二选一使用;
cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
condition：指定符合条件的情况下才缓存；
,condition = "#id>0"
condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
unless = "#result == null"
unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
sync：是否使用异步模式:默认为false
如果为true，即不支持unless属性

@Cacheable(value = {"emp"}) ==  @Cacheable(cacheNames = {"emp"})
运行时机：
1、在调用目标方法前，先到缓存中查找是否有缓存对应的值，如果有就去缓存中找，不执行目标方法。如果没有，执行2.
2、执行目标方法，并执行3
3、将查询的数据缓存起来。



**（3）、@CachePut**

```java
    /**
     * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时更新缓存；
     * 运行时机：
     *  1、先调用目标方法
     *  2、将目标方法的结果缓存起来
     *
     * 测试步骤：
     *  1、查询1号员工；查到的结果会放在缓存中；
     *          key：1  value：lastName：张三
     *  2、以后查询还是之前的结果
     *  3、更新1号员工；【lastName:zhangsan；gender:0】
     *          将方法的返回值也放进缓存了；
     *          key：传入的employee对象  值：返回的employee对象；
     *  4、查询1号员工？
     *      应该是更新后的员工；
     *          key = "#employee.id":使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *             @Cacheable的key是不能用#result
     *      为什么是没更新前的？【1号员工没有在缓存中更新】
     *
     */
    //,key = "#result.id"
    @CachePut(value = "emp",key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }
```



@CachePut：既调用方法，又更新缓存数据；同步更新缓存
修改了数据库的某个数据，同时更新缓存；
运行时机：
1、先调用目标方法
2、将目标方法的结果缓存起来
测试步骤：
1、查询1号员工；查到的结果会放在缓存中；
key：1  value：lastName：张三
2、以后查询还是之前的结果
3、更新1号员工；【lastName:zhangsan；gender:0】
将方法的返回值也放进缓存了；
key：传入的employee对象  值：返回的employee对象；
4、查询1号员工？
应该是更新后的员工；
key = "#employee.id":使用传入的参数的员工id；
key = "#result.id"：使用返回后的id
@Cacheable的key是不能用#result
为什么是没更新前的？【1号员工没有在缓存中更新】



**(4)、@CacheEvict**

```
 /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *
     *  allEntries = true：指定清除这个缓存中所有的数据(一般不使用，全部清除）
     *
     *  beforeInvocation = false：缓存的清除是否在方法之前执行（默认）
     *      默认代表缓存清除操作是在方法（目标方法）执行之后执行;如果出现异常缓存就不会清除
     *
     *  beforeInvocation = true：
     *      代表清除缓存操作是在方法（目标方法）运行之前执行，无论方法是否出现异常，缓存都清除
     *
     */
    //@CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    @CacheEvict(value="emp",key = "#id",beforeInvocation = true)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        //出异常
        int i = 10/0;
    }
```



@CacheEvict：缓存清除
key：指定要清除的数据
allEntries = true：指定清除这个缓存中所有的数据(一般不使用，全部清除）
beforeInvocation = false：缓存的清除是否在方法之前执行（默认）
默认代表缓存清除操作是在方法（目标方法）执行之后执行;如果出现异常缓存就不会清除
beforeInvocation = true：
代表清除缓存操作是在方法（目标方法）运行之前执行，无论方法是否出现异常，缓存都清除;



**(5)、@Caching**

```
// @Caching 定义复杂的缓存规则
    //@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置，类上面
    @Caching(
            cacheable = {
                    @Cacheable(/*value="emp",*/key = "#lastName")
            },
            put = {
                    @CachePut(/*value="emp",*/key = "#result.id"),
                    @CachePut(/*value="emp",*/key = "#result.email")
            }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }
```

@Caching 定义复杂的缓存规则



## 五、整合redis实现缓存

![1561961788499](.\assets\1561961788499.png)

当不引入redis的缓存实现时，boot引入默认的缓存。这个跟JDBC有点类似。引入对应数据库的驱动就相当于JDBC的实现。

**以上具体看springboot-cache这个项目**