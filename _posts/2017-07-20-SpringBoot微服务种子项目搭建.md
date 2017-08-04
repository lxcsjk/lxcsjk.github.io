---
layout: post
title:  "SpringBoot微服务种子项目搭建"
date:   2017-07-20 20:00:03 +0800
categories: jekyll update
tags: SpringBoot
---



公司系统架构目前是纯微服务调用， 相关技术栈：

- 前端：React + Webpack
- 后端：SpringBoot + Gradle + Docker

所以每次新添加一个功能都是一个新的微服务模块

为了快速开发，就自己弄了一种子项目，需要的时候拿来用。



先上个种子链接 [项目地址](https://github.com/lxcsjk/spring-boot-seed-project)

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170804-1.png)

So，开始吧。

---



### 开发环境



- 系统：macOS Sierra 10.12.5

- IDE  ：IntelliJ IDEA 2017.2.1

- JDK  ：Java(TM) SE Runtime Environment (build 1.8.0_141-b15)

- Gradle ： Gradle 4.0.1

  ​

---



### 新建项目



新建一个Gradle项目

​

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170803-1.png)

​

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170803-2.png)

​

我一般选择自动导入 = = 

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170803-3.png)

​

在读取索引和创建空文件夹

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170803-4.png)

​

由于是一个单独的模块 就删除了 **setting.gradle**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170803-6.png)

​

添加依赖的jar到 **build.gradle**

```groovy
buildscript {
    ext {
        springBootVersion = '1.5.6.RELEASE'
    }
    repositories {
        mavenLocal()
        //daoCloud的私服 下载jar更快
        maven { url "http://nexus.daocloud.io/repository/maven-public/" }
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'
apply plugin: 'idea'

jar {
    baseName = 'seed-service'
    version = '0.0.1-SNAPSHOT'
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenLocal()
    maven { url "http://nexus.daocloud.io/repository/maven-public/" }
    mavenCentral()
}

dependencies {
    // 处理java8新的时间类型
    compile group: 'com.fasterxml.jackson.datatype', name: 'jackson-datatype-jsr310', version: '2.9.0'

    // 用来查看服务运行的信息 生产环境可以在配置文件关闭
    compile('org.springframework.boot:spring-boot-starter-actuator')
    // 就是SpringMVC
    compile('org.springframework.boot:spring-boot-starter-web')
    // 打印日志框架
    compile('org.springframework.boot:spring-boot-starter-logging')
    // 消息重试
    compile('org.springframework.retry:spring-retry')

    // 工具类
    compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.6'
    // Guava google出的非常棒的工具类
    compile group: 'com.google.guava', name: 'guava', version: '22.0'
    // 注解生成get set方法等 需要项目打开 Annotation Processors
    compile group: 'org.projectlombok', name: 'lombok', version: '1.16.18'

    // 数据库驱动
    compile group: 'mysql', name: 'mysql-connector-java', version: '6.0.6'

    // 数据库连接池
    compile group: 'com.zaxxer', name: 'HikariCP', version: '2.6.3'

    // mybatis
    compile group: 'org.mybatis.spring.boot', name: 'mybatis-spring-boot-starter', version: '1.3.0'
    // mybatis 类型转换器
    compile group: 'org.mybatis', name: 'mybatis-typehandlers-jsr310', version: '1.0.2'
    // mybatis 分页插件
    compile group: 'com.github.pagehelper', name: 'pagehelper-spring-boot-starter', version: '1.1.2'
    // mybatis 通用Mapper 像jpa一样好用
    compile group: 'tk.mybatis', name: 'mapper-spring-boot-starter', version: '1.1.2'
    // 数据库结构同步
    compile group: 'org.flywaydb', name: 'flyway-core', version: '4.2.0'

    // HttpClient retrofit2
    compile group: 'com.squareup.retrofit2', name: 'retrofit', version: '2.3.0'
    compile group: 'com.squareup.retrofit2', name: 'converter-gson', version: '2.3.0'
    compile group: 'com.squareup.okhttp3', name: 'okhttp', version: '3.8.1'
    compile group: 'com.squareup.okhttp3', name: 'logging-interceptor', version: '3.8.1'

    // RxJava
    compile group: 'io.reactivex.rxjava2', name: 'rxjava', version: '2.1.2'

    // 测试
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

​

添加这些jar也是根据自己的喜好，当然你也可以选择去掉一些你用不到的jar。

​

**IDEA** 打开 **Annotation Processors**

![](http://oh6uhie7j.bkt.clouddn.com/QQ20170803-7.png)

新建一个项目差不多完成了。然后开始配置一些Bean。

---



### 配置多数据源



1. **yml** 文件

   ```yaml
   spring:
     application:
       name: seed-service
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       type: com.zaxxer.hikari.HikariDataSource
       hikari:
         connection-test-query: SELECT 1
         maximum-pool-size: 20
         minimum-idle: 2
         max-lifetime: 60000
       primary:
         jdbc-url: jdbc:mysql://localhost:3306/test1?characterEncoding=UTF-8&useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull&useSSL=false&allowMultiQueries=true
         username: root
         password: root
       second:
         jdbc-url: jdbc:mysql://localhost:3306/test2?characterEncoding=UTF-8&useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull&useSSL=false&allowMultiQueries=true
         username: root
         password: root
     profiles:
       active: dev

   logging:
     config: classpath:logback.xml

   mybatis:
     type-aliases-package: com.betterlxc.mvc.domain
     mapper-locations: classpath:mapper/*.xml

   mapper:
     mappers: com.betterlxc.utils.BaseMapper
     not-empty: false
     identity: MYSQL
   ```

   配置了两个数据 一个为主 一个为辅 ，这种配置方式适合数据源不多的情况，如果很多可以使用**@注解** 基于**AOP**，需在 **bulid.gradle** 中添加 aop的依赖，以后另起一篇详细描述关于多数据源配置的问题。

   ​

   **DataSourcePrimaryConfig**

   ```java
   package com.betterlxc.config.datasource;

   import com.zaxxer.hikari.HikariDataSource;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.SqlSessionFactoryBean;
   import org.mybatis.spring.SqlSessionTemplate;
   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.Primary;
   import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
   import org.springframework.jdbc.datasource.DataSourceTransactionManager;

   /**
    * Created by LXC on 2017/5/10.
    */
   @Configuration
   @MapperScan(basePackages = DataSourcePrimaryConfig.PACKAGE, sqlSessionTemplateRef = "primarySqlSessionTemplate")
   public class DataSourcePrimaryConfig {

     static final String PACKAGE = "com.betterlxc.mvc.mapper.primary";

     @Bean(name = "primaryDataSource")
     @Primary
     @ConfigurationProperties(prefix = "spring.datasource.primary")
     public HikariDataSource primaryDataSourceDataSource() {
       return new HikariDataSource();
     }

     @Bean(name = "primaryTransactionManager")
     @Primary
     public DataSourceTransactionManager primaryDataSourceTransactionManager() {
       return new DataSourceTransactionManager(primaryDataSourceDataSource());
     }

     @Bean(name = "primarySqlSessionFactory")
     @Primary
     public SqlSessionFactory primaryDataSourceSqlSessionFactory(
         @Qualifier("primaryDataSource") HikariDataSource primaryDataSourceDataSource) throws Exception {
       final SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
       bean.setDataSource(primaryDataSourceDataSource);
       bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/primary/*.xml"));
       return bean.getObject();
     }

     @Bean(name = "primarySqlSessionTemplate")
     @Primary
     public SqlSessionTemplate primarySqlSessionTemplate(@Qualifier("primarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
       return new SqlSessionTemplate(sqlSessionFactory);
     }
   }
   ```

   ​

   **DataSourceSecondConfig**

   ```java
   package com.betterlxc.config.datasource;

   import com.zaxxer.hikari.HikariDataSource;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.SqlSessionFactoryBean;
   import org.mybatis.spring.SqlSessionTemplate;
   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
   import org.springframework.jdbc.datasource.DataSourceTransactionManager;

   /**
    * Created by LXC on 2017/5/10.
    */
   @Configuration
   @MapperScan(basePackages = DataSourceSecondConfig.PACKAGE, sqlSessionTemplateRef = "secondSqlSessionTemplate")
   public class DataSourceSecondConfig {

     static final String PACKAGE = "com.betterlxc.mvc.mapper.second";

     @Bean(name = "secondDataSource")
     @ConfigurationProperties(prefix = "spring.datasource.second")
     public HikariDataSource secondDataSourceDataSource() {
       return new HikariDataSource();
     }

     @Bean(name = "secondTransactionManager")
     public DataSourceTransactionManager secondDataSourceTransactionManager() {
       return new DataSourceTransactionManager(secondDataSourceDataSource());
     }

     @Bean(name = "secondSqlSessionFactory")
     public SqlSessionFactory secondDataSourceSqlSessionFactory(
         @Qualifier("secondDataSource") HikariDataSource secondDataSourceDataSource) throws Exception {
       final SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
       bean.setDataSource(secondDataSourceDataSource);
       bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/second/*.xml"));
       return bean.getObject();
     }

     @Bean(name = "secondSqlSessionTemplate")
     public SqlSessionTemplate secondSqlSessionTemplate(@Qualifier("secondSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
       return new SqlSessionTemplate(sqlSessionFactory);
     }
   }
   ```

   注意**Mybatis**文件的路径 如果是**Jpa**或者**Groovy**就不要指定MapperLocations啦




---



### 统一异常处理



```java
package com.betterlxc;

import com.google.common.base.Throwables;
import com.google.common.collect.ImmutableMap;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import java.util.Map;

/**
 * Created by LXC on 2017/5/10.
 */
@Slf4j
@ControllerAdvice(basePackages = "com.betterlxc.mvc.controller")
public class GlobalControllerExceptionHandler {

  @ResponseBody
  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  @ExceptionHandler({SeedException.class})
  public Map<String, String> handleException(Throwable e) {
    log.warn("", e);

    return ImmutableMap.of("msg",
        String.valueOf(Throwables.getRootCause(e).getMessage()));
  }
}
```



---



### 注入常用实体



**DefaultConfiguration** 

- 对视图解析器进行配置
- httpClient添加自定义解析器和拦截器
- **Flyway**初始化（不过建议Flyway在确定了表结构在加入）
- 添加过滤器

```java
package com.betterlxc.config;

import com.betterlxc.client.Interceptors;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.PropertyNamingStrategy;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.zaxxer.hikari.HikariDataSource;
import okhttp3.OkHttpClient;
import okhttp3.logging.HttpLoggingInterceptor;
import org.flywaydb.core.Flyway;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.jackson.Jackson2ObjectMapperBuilderCustomizer;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;

import java.time.LocalDateTime;
import java.util.concurrent.TimeUnit;

/**
 * Created by GuoTianxiang on 2016/11/12.
 */
@Configuration
public class DefaultConfiguration {

  @Order(1)
  @Bean("mbxcDefaultJackson2ObjectMapperBuilderCustomizer")
  @ConditionalOnClass({ObjectMapper.class, Jackson2ObjectMapperBuilder.class})
  public Jackson2ObjectMapperBuilderCustomizer mbxcDefaultJackson2ObjectMapperBuilderCustomizer() {
    return builder -> {
      JavaTimeModule module = new JavaTimeModule();
      module.addDeserializer(LocalDateTime.class, MyLocalDateTimeDeserializer.INSTANCE);

      builder.modules(module);
      builder.featuresToDisable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
      builder.featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
      builder.featuresToDisable(SerializationFeature.WRITE_NULL_MAP_VALUES);
      builder.propertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
      builder.serializationInclusion(JsonInclude.Include.NON_NULL);
    };
  }

  @Bean
  @ConditionalOnClass(ObjectMapper.class)
  public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
    return new MappingJackson2HttpMessageConverter(objectMapper);
  }

  @Bean
  @ConditionalOnClass({OkHttpClient.class, HttpLoggingInterceptor.class})
  @ConditionalOnMissingBean(name = "httpLoggingInterceptor")
  public HttpLoggingInterceptor httpLoggingInterceptor(@Value("${http.level:BASIC}") String level) {
    HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
    interceptor.setLevel(HttpLoggingInterceptor.Level.valueOf(level.toUpperCase()));
    return interceptor;
  }

  @Bean
  @ConditionalOnClass({OkHttpClient.class, HttpLoggingInterceptor.class})
  @ConditionalOnMissingBean(name = "okHttpClient")
  public OkHttpClient okHttpClient(HttpLoggingInterceptor httpLoggingInterceptor) {
    return new OkHttpClient.Builder()
        .readTimeout(5, TimeUnit.MINUTES)
        .addInterceptor(httpLoggingInterceptor)
        .addInterceptor(Interceptors.httpHeader("X-BETTER-LXC", "YoRuo"))
        .addInterceptor(Interceptors::errorLog)
        .build();
  }

  @Bean
  @ConditionalOnMissingBean
  public FilterRegistrationBean filterRegistrationBean() {
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new LogFilter());
    bean.setOrder(Ordered.LOWEST_PRECEDENCE);
    return bean;
  }

  @Bean(initMethod = "migrate", name = "primaryFlyway")
  public Flyway primaryFlyway(@Qualifier("primaryDataSource") HikariDataSource hikariDataSource) {
    Flyway primaryFlyway = new Flyway();
    primaryFlyway.setDataSource(hikariDataSource);
    return primaryFlyway;
  }

  @Bean(initMethod = "migrate", name = "secondFlyway")
  public Flyway secondFlyway(@Qualifier("secondDataSource") HikariDataSource hikariDataSource) {
    Flyway secondFlyway = new Flyway();
    secondFlyway.setDataSource(hikariDataSource);
    return secondFlyway;
  }
}
```



---



### 启动项目



**Main**方法启动看日志。



![](http://oh6uhie7j.bkt.clouddn.com/QQ20170804-3.png)



---



差不多就是这些，具体的下载项目后看吧。

会不断学习并更新这个种子项目~~~ 

= = 






