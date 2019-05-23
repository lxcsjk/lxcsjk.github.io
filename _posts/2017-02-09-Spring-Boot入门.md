---
layout: post
title:  "Spring-Boot入门"
date:   2017-02-09 10:00:03 +0800
categories: jekyll update
tags: Spring
---

之前项目中有个服务需要独立出来，同事用springboot改成了微服务，这两天没啥忙的就给拿出瞅瞅学习一下~

Demo只是对 这个[例子](https://github.com/abel533/MyBatis-Spring-Boot) 重新写了一遍加了一些注释和自己的东西 = =

---

### 新建一个maven项目

我用的是IDEA，新建maven项目时，记得添加一个属性 **archetypeCatalog=internal** 

> archetypeCatalog表示插件使用的archetype元数据，不加这个参数时默认为remote，local，即中央仓库archetype元数据，由于中央仓库的archetype太多了，所以导致很慢，指定internal来表示仅使用内部元数据

![](http://lxc.xiaocblog.com/QQ20170209-191321@2x.png)

![](http://lxc.xiaocblog.com/QQ20170209-191419@2x.png)

![](http://lxc.xiaocblog.com/QQ20170209-191425@2x.png)

---

### 添加配置

在**pom.xml**文件添加响应的配置

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-tools</artifactId>
		<version>1.5.1.RELEASE</version>
	</parent>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<name>Spring Boot Configuration Processor</name>
	<description>Spring Boot Configuration Processor</description>
	<url>http://projects.spring.io/spring-boot/</url>
	<organization>
		<name>Pivotal Software, Inc.</name>
		<url>http://www.spring.io</url>
	</organization>
	<properties>
		<main.basedir>${basedir}/../..</main.basedir>
	</properties>
	<dependencies>
 		<!-- Compile (should stick to the bare minimum) -->
 		<dependency>
			<groupId>com.vaadin.external.google</groupId>
			<artifactId>android-json</artifactId>
		</dependency>
		<!-- Test -->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-test-support</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<!-- Ensure own annotation processor doesn't kick in -->
					<proc>none</proc>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

在application.properties中写入相关配置

```
server.port=9000

druid.url=jdbc:mysql://127.0.0.1:3306/test
druid.username=root
druid.password=root
druid.initialSize=1
druid.minIdle=1
druid.maxActive=20
druid.testOnBorrow=true

#springboot默认的日志系统是logback
logging.level.tk.mybatis=TRACE

#freemarker 文件的路径
spring.mvc.view.prefix=/templates/
spring.mvc.view.suffix=.ftl
spring.freemarker.cache=false
spring.freemarker.request-context-attribute=request

#mybatis扫描的文件路径
mybatis.type-aliases-package=tk.yoruo.springboot.model
mybatis.mapper-locations=classpath:mapper/*.xml

#通用mapper的配置
mapper.mappers=tk.yoruo.springboot.util.MyMapper
mapper.not-empty=false
mapper.identity=MYSQL

#mybatis分页插件的配置
pagehelper.helper-dialect=mysql
pagehelper.reasonable=true
pagehelper.support-methods-arguments=true
pagehelper.params=count=countSql

```

---

### 静态资源的路径

**用了freemarker模版引擎 配置一些静态资源文件路径**

```
package tk.yoruo.springboot.conf;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

/**
 * Created by LXC on 2017/2/9.
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
```

其他的service层，实体，controller就不阐述了。

---

### 添加数据库配置

创建一个**DruidProperties**的类

添加一些属性。并加上**@ConfigurationProperties(prefix = "druid")**注解，告诉boot这个是个读取配置文件的类并且从 **application.properties** 中取前缀为 **druid**的value。

接着创建**DruidAutoConfiguration**类 加上@Configuration告诉boot这个是配置类。

注入DruidProperties 并加入**@Bean**

```
package tk.yoruo.springboot.druid;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.sql.SQLException;

/**
 * Created by LXC on 2017/2/9.
 */

@Configuration
@EnableConfigurationProperties(DruidProperties.class)
// 对应的类在classpath中有存在时才会注入
@ConditionalOnClass(DruidDataSource.class)

// 对应的前缀和属性名 在classpath中有存在时才会注入
@ConditionalOnProperty(prefix = "druid", name = "url")
public class DruidAutoConfiguration {

    @Autowired
    private DruidProperties druidProperties;

    @Bean
    public DruidDataSource dataSource() {

        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(druidProperties.getUrl());
        druidDataSource.setUsername(druidProperties.getUsername());
        druidDataSource.setPassword(druidProperties.getPassword());
        if (druidProperties.getInitialSize() > 0) {
            druidDataSource.setInitialSize(druidProperties.getInitialSize());
        }
        if (druidProperties.getMinIdle() > 0) {
            druidDataSource.setMinIdle(druidProperties.getMinIdle());
        }
        if (druidProperties.getMaxActive() > 0) {
            druidDataSource.setMaxActive(druidProperties.getMaxActive());
        }
        druidDataSource.setTestOnBorrow(druidProperties.isTestOnBorrow());
        try {
            druidDataSource.init();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        return druidDataSource;
    }
}
```

接着 在src/main/resources目录下新建META-INF文件夹，然后新建spring.factories文件，这个文件用于告诉Spring Boot去找指定的自动配置文件

所以spring.factories内容为

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
tk.yoruo.springboot.druid.DruidAutoConfiguration
```

---

### 添加Druid Web监控

分别继承 **Druid**中的过滤器和servlet。

```
package tk.yoruo.springboot.druid;

import com.alibaba.druid.support.http.WebStatFilter;

import javax.servlet.annotation.WebFilter;
import javax.servlet.annotation.WebInitParam;

/**
 * Created by LXC on 2017/2/8.
 */

@WebFilter(filterName = "druidWebStatFilter", urlPatterns = "/*",
        initParams = {
                @WebInitParam(name = "exclusions", value = "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")// 忽略资源
        })
public class DruidStatFilter extends WebStatFilter {

}


```

```
package tk.yoruo.springboot.druid;

import com.alibaba.druid.support.http.StatViewServlet;

import javax.servlet.annotation.WebInitParam;
import javax.servlet.annotation.WebServlet;

/**
 * Created by LXC on 2017/2/8.
 */

@SuppressWarnings("serial")
@WebServlet(urlPatterns = "/druid/*",
        initParams = {
                @WebInitParam(name = "allow", value = "127.0.0.1"),// IP白名单 (没有配置或者为空，则允许所有访问)
//                @WebInitParam(name="deny",value="192.168.16.111"),// IP黑名单 (存在共同时，deny优先于allow)
                @WebInitParam(name = "loginUsername", value = "admin"),// 用户名
                @WebInitParam(name = "loginPassword", value = "admin"),// 密码
                @WebInitParam(name = "resetEnable", value = "true")// 禁用HTML页面上的“Reset All”功能
        })
public class DruidStatViewServlet extends StatViewServlet {

}

```

#### 注意还需要在Application类上加入 **@ServletComponentScan**的注解，否则扫描不到你添加的servlet。

```
package tk.yoruo.springboot;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import tk.yoruo.springboot.util.MyMapper;

/**
 * Created by LXC on 2017/2/9.
 */


@Controller
@ServletComponentScan
@EnableWebMvc
//@ComponentScan(basePackages = {"tk.yoruo.springboot.controller","tk.yoruo.springboot.service"})
@SpringBootApplication
@MapperScan(basePackages = "tk.yoruo.springboot.mapper", markerInterface = MyMapper.class)
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @RequestMapping("/")
    String home() {
        return "redirect:countries";
    }

}

```

---

### 注意

**Application** springboot启动的类，不建议放在顶级的包。例如**src/main/java**下面。

如果这样做了会和我一样出现这个错误

>** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.

---

如果一定要这样做，需要在**Application**上加这样的注解。标明你要扫描的包

```
@ComponentScan(basePackages = {"tk.yoruo.springboot.controller","tk.yoruo.springboot.service"})
```
这样话静态资源会找不到。

那应该怎么找到... 我还没去试 = = 

建议别这样做，放在外面启动服务时间会变久。

---

