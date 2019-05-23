---
layout: post
title:  "SpringBoot构建Web项目"
date:   2017-04-05 23:30:03 +0800
categories: jekyll update
tags: Spring
---



### 写在前面

最近需要重构一个项目，所以就去研究了关于SpringBoot搭建web的东西。基本的技术栈

- 基础的服务框架 ：``SpringBoot 1.5.2``

- 持久层的框架：``MyBatis 3.4+``   本来使用的是``Jpa``，写了一会发现会有很多关联查询和分组就改成了MyBatis，之前写了一个 ``Mybatis Geneator``代码生成器所以写起来也很快。

- 构建工具  ``Gradle 3.0``

- 前端框架  ``Angular.js``

  ​

### 开始搭建环境

``build.gradle`` 添加相关依赖

````groovy
buildscript {
    ext {
        springBootVersion = '1.5.2.RELEASE'
    }
    repositories {
        mavenLocal()
        maven { url "http://nexus.chachazhan.com/content/groups/public/" }
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
    baseName = 'springboot-web-jsp'
    version = '0.0.1-SNAPSHOT'
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenLocal()
    maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
````



添加相关的类 ，项目结构如下

![](http://lxc.xiaocblog.com/QQ20170405-2.png)



**添加  ``WebMvcConfig``  配置外部静态资源**



````java
package com.betterlxc.web.demo.conf;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

/**
 * Created by LXC on 2017/3/28.
 */
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

  /**
   * spring-boot配置外部静态资源的方法
   *
   * @param registry
   */
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
  }
}

````



启动后访问  [http://localhost:8080/](http://localhost:8080/)

**报错 信息如下**

```java
javax.servlet.ServletException: Could not resolve view with name 'index' in servlet with name 'dispatcherServlet'
	at org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1262) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1037) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:980) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:897) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:861) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:622) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:729) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:230) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:165) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52) ~[tomcat-embed-websocket-8.5.11.jar:8.5.11]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:192) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:165) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:197) ~[spring-web-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:192) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:165) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
...
```



在``application.properties``里已经配置

```properties
server.port=8080

spring.mvc.view.prefix=/static/
spring.mvc.view.suffix=.jsp
```



**那么问题来了...  为啥会这样呢 静态资源的访问路径也配置了。 那肯定是视图解析器出了问题，我们来配置一下视图解析器吧**



在``conf``目录下创建一个配置类``DefaultConfiguration``



```java
package com.betterlxc.web.demo.conf;


import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.view.JstlView;
import org.springframework.web.servlet.view.UrlBasedViewResolver;

/**
 * Created by LXC on 2017/3/28.
 */
@Configuration
public class DefaultConfiguration {

  @Bean
  public UrlBasedViewResolver setupViewResolver() {
    UrlBasedViewResolver resolver = new UrlBasedViewResolver();
    resolver.setPrefix("/static/");
    resolver.setSuffix(".jsp");
    resolver.setCache(true);
    resolver.setViewClass(JstlView.class);
    return resolver;
  }
}
```



重新 **run-bulid**

刷新浏览器， 还是报错了但是错误不同。

```java
java.lang.ClassNotFoundException: javax.servlet.jsp.jstl.core.Config
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381) ~[na:1.8.0_121]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_121]
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331) ~[na:1.8.0_121]
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_121]
	at org.springframework.web.servlet.support.JstlUtils.exposeLocalizationContext(JstlUtils.java:101) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.view.JstlView.exposeHelpers(JstlView.java:135) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:142) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:303) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1282) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1037) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:980) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:897) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:861) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:622) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846) ~[spring-webmvc-4.3.7.RELEASE.jar:4.3.7.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:729) ~[tomcat-embed-core-8.5.11.jar:8.5.11]
	at .....
```



添加一下 缺少的jar包

```groovy
compile group: 'javax.servlet', name: 'jstl', version: '1.2'
```



重新run

再次访问 发现直接下载了 **因为浏览器是不认识jsp中的java代码的，jsp是需要先编译成servlet和html。**



决定去看一下springboot文档，发现文档中很明确写到了：



> JSP limitations
>
> When running a Spring Boot application that uses an embedded servlet container (and is packaged asan executable archive), there are some limitations in the JSP support.
>
> - With Tomcat it should work if you use war packaging, i.e. an executable war will work, and will alsobe deployable to a standard container (not limited to, but including Tomcat). An executable jar will notwork because of a hard coded file pattern in Tomcat.
>
> - With Jetty it should work if you use war packaging, i.e. an executable war will work, and will also bedeployable to any standard container.
>
> - Undertow does not support JSPs.
>
> - Creatingacustomerror.jsppagewon’toverridethedefaultviewforerrorhandling,customerror
>
>   pages should be used instead.
>   There is a JSP sample so you can see how to set things up.

​		

在[github](https://github.com/)还提供了一个[jsp示例](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-jsp)



---



### 改动



修改  **build.gradle**

```groovy
buildscript {
  ext {
    springBootVersion = '1.5.2.RELEASE'
  }
  repositories {
    mavenLocal()
    maven { url "http://nexus.chachazhan.com/content/groups/public/" }
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'war'
apply plugin: 'spring-boot'
apply plugin: 'idea'

war {
  baseName = 'springboot-web-jsp'
  version = '0.0.1-SNAPSHOT'
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
  mavenLocal()
  maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile group: 'javax.servlet', name: 'jstl', version: '1.2'
  compile group: 'org.apache.tomcat.embed', name: 'tomcat-embed-jasper', version: '8.5.11'
  compile('org.springframework.boot:spring-boot-starter-tomcat')

  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```



``WebApplication``类 继承 ``SpringBootServletInitializer``

```java
package com.betterlxc.web.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.stereotype.Controller;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

/**
 * Created by LXC on 2017/3/28.
 */


@Controller
@EnableWebMvc
@ComponentScan(basePackages = {"com.betterlxc.web.demo.conf", "com.betterlxc.web.demo.mvc.controller"})
@SpringBootApplication
public class WebApplication extends SpringBootServletInitializer {

  public static void main(String[] args) {
    SpringApplication.run(WebApplication.class, args);
  }

  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
    return application.sources(WebApplication.class);
  }
}
```



使用 war的插件后 目录结构会发生一些变化

![](http://lxc.xiaocblog.com/QQ20170405-3.png)



重新访问   [http://localhost:8080/](http://localhost:8080/)



![](http://lxc.xiaocblog.com/QQ20170405-4.png)





**SpringBoot其实不推荐使用jsp，可使用其他的模版引擎，例如[freemarker](http://freemarker.org/) ,[Beetl](http://www.ibeetl.com/guide/#beetl)**



[demo源码](https://github.com/lxcsjk/springboot-web-jsp)



