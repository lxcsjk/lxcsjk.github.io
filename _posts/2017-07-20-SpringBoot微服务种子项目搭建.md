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



1. 新建一个Gradle项目

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



### 相关配置

