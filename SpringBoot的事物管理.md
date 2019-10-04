#  事物管理

##  一、SpringBoot整合事物管理

Springboot默认集成事物,只主要在方法上加上@Transactional即可



## 二、SpringBoot分布式事物管理

使用springboot+jta+atomikos 分布式事物管理

Atomikos 是一个为Java平台提供增值服务的并且开源类事务管理器。

![1570158592405](.\assets\1570158592405.png)



### 2.1、新增jta-atomikos依赖



```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jta-atomikos</artifactId>
        </dependency>
```



### 2.2、新增配置文件信息

```properties
# Mysql 1
mysql.datasource.test1.url = jdbc:mysql://localhost:3306/itmy_test01?useUnicode=true&characterEncoding=utf-8
mysql.datasource.test1.username = root
mysql.datasource.test1.password = root
mysql.datasource.test1.minPoolSize = 3
mysql.datasource.test1.maxPoolSize = 25
mysql.datasource.test1.maxLifetime = 20000
mysql.datasource.test1.borrowConnectionTimeout = 30
mysql.datasource.test1.loginTimeout = 30
mysql.datasource.test1.maintenanceInterval = 60
mysql.datasource.test1.maxIdleTime = 60



# Mysql 2
mysql.datasource.test2.url =jdbc:mysql://localhost:3306/itmy_test02?useUnicode=true&characterEncoding=utf-8
mysql.datasource.test2.username =root
mysql.datasource.test2.password =root
mysql.datasource.test2.minPoolSize = 3
mysql.datasource.test2.maxPoolSize = 25
mysql.datasource.test2.maxLifetime = 20000
mysql.datasource.test2.borrowConnectionTimeout = 30
mysql.datasource.test2.loginTimeout = 30
mysql.datasource.test2.maintenanceInterval = 60
mysql.datasource.test2.maxIdleTime = 60

```



### 2.3、读取配置文件信息（两个配置类）

#### DBConfig1.java

```java
package com.nieshenkuan.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 22:42
 */
@Data
@ConfigurationProperties(prefix = "mysql.datasource.test1")
public class DBConfig1 {

    private String url;
    private String username;
    private String password;
    private int minPoolSize;
    private int maxPoolSize;
    private int maxLifetime;
    private int borrowConnectionTimeout;
    private int loginTimeout;
    private int maintenanceInterval;
    private int maxIdleTime;
    private String testQuery;
}

```



#### DBConfig2.java

```java
package com.nieshenkuan.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 22:43
 */
@Data
@ConfigurationProperties(prefix = "mysql.datasource.test2")
public class DBConfig2 {

    private String url;
    private String username;
    private String password;
    private int minPoolSize;
    private int maxPoolSize;
    private int maxLifetime;
    private int borrowConnectionTimeout;
    private int loginTimeout;
    private int maintenanceInterval;
    private int maxIdleTime;
    private String testQuery;
}
```



### 2.4、创建多数据源



#### MyBatisConfig1.java

```java
package com.nieshenkuan.datasource;
import java.sql.SQLException;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.atomikos.jdbc.AtomikosDataSourceBean;
import com.nieshenkuan.config.DBConfig1;
import com.mysql.jdbc.jdbc2.optional.MysqlXADataSource;
/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 22:46
 */
@Configuration
// basePackages 最好分开配置 如果放在同一个文件夹可能会报错
@MapperScan(basePackages = "com.nieshenkuan.test01", sqlSessionTemplateRef = "testSqlSessionTemplate")
public class MyBatisConfig1 {

    // 配置数据源

    @Bean(name = "testDataSource")
    public DataSource testDataSource(DBConfig1 testConfig) throws SQLException {
        MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
        mysqlXaDataSource.setUrl(testConfig.getUrl());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
        mysqlXaDataSource.setPassword(testConfig.getPassword());
        mysqlXaDataSource.setUser(testConfig.getUsername());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);

        // 将本地事务注册到创 Atomikos全局事务 (只用一个全局事务。多个数据源用同一个事务，将每一个数据源整合到这个事务中）
        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(mysqlXaDataSource);
        xaDataSource.setUniqueResourceName("testDataSource");

        xaDataSource.setMinPoolSize(testConfig.getMinPoolSize());
        xaDataSource.setMaxPoolSize(testConfig.getMaxPoolSize());
        xaDataSource.setMaxLifetime(testConfig.getMaxLifetime());
        xaDataSource.setBorrowConnectionTimeout(testConfig.getBorrowConnectionTimeout());
        xaDataSource.setLoginTimeout(testConfig.getLoginTimeout());
        xaDataSource.setMaintenanceInterval(testConfig.getMaintenanceInterval());
        xaDataSource.setMaxIdleTime(testConfig.getMaxIdleTime());
        xaDataSource.setTestQuery(testConfig.getTestQuery());
        return xaDataSource;
    }

    @Bean(name = "testSqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("testDataSource") DataSource dataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean.getObject();
    }

    @Bean(name = "testSqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("testSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```



#### MyBatisConfig2.java

```java
package com.nieshenkuan.datasource;
import java.sql.SQLException;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.atomikos.jdbc.AtomikosDataSourceBean;
import com.nieshenkuan.config.DBConfig2;
import com.mysql.jdbc.jdbc2.optional.MysqlXADataSource;
/**
 * @Description:
 * @author: nieshenkuan
 * @Date: 2019/10/1 22:47
 */
@Configuration
@MapperScan(basePackages = "com.nieshenkuan.test02", sqlSessionTemplateRef = "test2SqlSessionTemplate")
public class MyBatisConfig2 {

    // 配置数据源
    @Bean(name = "test2DataSource")
    public DataSource testDataSource(DBConfig2 testConfig) throws SQLException {
        MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
        mysqlXaDataSource.setUrl(testConfig.getUrl());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
        mysqlXaDataSource.setPassword(testConfig.getPassword());
        mysqlXaDataSource.setUser(testConfig.getUsername());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);

        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(mysqlXaDataSource);
        xaDataSource.setUniqueResourceName("test2DataSource");

        xaDataSource.setMinPoolSize(testConfig.getMinPoolSize());
        xaDataSource.setMaxPoolSize(testConfig.getMaxPoolSize());
        xaDataSource.setMaxLifetime(testConfig.getMaxLifetime());
        xaDataSource.setBorrowConnectionTimeout(testConfig.getBorrowConnectionTimeout());
        xaDataSource.setLoginTimeout(testConfig.getLoginTimeout());
        xaDataSource.setMaintenanceInterval(testConfig.getMaintenanceInterval());
        xaDataSource.setMaxIdleTime(testConfig.getMaxIdleTime());
        xaDataSource.setTestQuery(testConfig.getTestQuery());
        return xaDataSource;
    }

    @Bean(name = "test2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test2DataSource") DataSource dataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean.getObject();
    }

    @Bean(name = "test2SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("test2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```



### 2.5、启动加载配置

```java
// 开启读取配置文件
@EnableConfigurationProperties(value = { DBConfig1.class, DBConfig2.class })
```





## 总结

![1570158592405](.\assets\1570158592405.png)

如上图，多数据源的多个事务都整合为一个全局事务。多个数据源用同一个全局事务，这样就不会出现事务冲突问题了。

**具体看itmy-springboot-datasource-jta项目。**