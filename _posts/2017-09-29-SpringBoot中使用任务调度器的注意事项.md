---
layout: post
title:  "SpringBoot中使用任务调度器的注意事项"
date:   2017-09-29 19:00:03 +0800
categories: jekyll update
tags: SpringBoot Schedule
---

---

很久没用写过blog，在忙一些事情，丢下了很多需要学习的时间。

正好十一提前回来了，就捡起来重新开始，毕竟，目标还是那么的遥不可及。

---



前段时间遇到了一个问题，一个服务的Task任务时间久了会整个服务休眠，也并没有发生异常，但是有**OOM**(具体原因也会在之后的blog总结)。只能重启解决问题了，然后开始一点点的Review代码。

最后找到了，对Spring的**@Schedule**的理解不够透彻，故把文档刷了一遍。

---



### @Schedule的value

@Schedule() 内的参数类型有好几种，分别代表的作用和意思有所不同，运用在不同的场景效果也会不一样。

见属性上的注释

```java
package org.springframework.scheduling.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * An annotation that marks a method to be scheduled. Exactly one of
 * the {@link #cron()}, {@link #fixedDelay()}, or {@link #fixedRate()}
 * attributes must be specified.
 * 
 在scheduled内，key只能填这三种 cron fixedDelay fixedRate
 *
 * <p>The annotated method must expect no arguments. It will typically have
 * a {@code void} return type; if not, the returned value will be ignored
 * when called through the scheduler.
 *
 所修饰的方法体上必须没有参数  通常返回的类型为 关键字void 
 如果不是 当调用这个任务调度器时返回值将被忽略
 *
 * <p>Processing of {@code @Scheduled} annotations is performed by
 * registering a {@link ScheduledAnnotationBeanPostProcessor}. This can be
 * done manually or, more conveniently, through the {@code <task:annotation-driven/>}
 * element or @{@link EnableScheduling} annotation.
 *
 @Scheduled 处理这个注解是在 ScheduledAnnotationBeanPostProcessor 这个类中进行注册 
 需要手动的去完成这个动作，在配置文件中添加 <task:annotation-driven/> 或者 添加 @EnableScheduling注解
 *
 * <p>This annotation may be used as a <em>meta-annotation</em> to create custom
 * <em>composed annotations</em> with attribute overrides.
 *
 这个注解可以作为元注解进行自由的扩展 对其它annotation类型作说明
 *
 * @author Mark Fisher
 * @author Dave Syer
 * @author Chris Beams
 * @since 3.0
 * @see EnableScheduling
 * @see ScheduledAnnotationBeanPostProcessor
 * @see Schedules
 */
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(Schedules.class)
public @interface Scheduled {
  /**
   * A cron-like expression, extending the usual UN*X definition to include
   * triggers on the second as well as minute, hour, day of month, month
   * and day of week.  e.g. {@code "0 * * * * MON-FRI"} means once per minute on
   * weekdays (at the top of the minute - the 0th second).
   * @return an expression that can be parsed to a cron schedule
   * @see org.springframework.scheduling.support.CronSequenceGenerator
   */


  继承于Unix的cron触发器
  
  String cron() default "";

  /**
   * A time zone for which the cron expression will be resolved. By default, this
   * attribute is the empty String (i.e. the server's local time zone will be used).
   *
   * @return a zone id accepted by {@link java.util.TimeZone#getTimeZone(String)},
   * or an empty String to indicate the server's default time zone
   * @see org.springframework.scheduling.support.CronTrigger#CronTrigger(String, java.util.TimeZone)
   * @see java.util.TimeZone
   * @since 4.0
   */

 
  指定一个cron触发器的时区 java.util.TimeZone
    
  String zone() default "";

  /**
   * Execute the annotated method with a fixed period in milliseconds between the
   * end of the last invocation and the start of the next.
   *
   * @return the delay in milliseconds
   */

  
   在最后一次调用的结束和下一个调用的开始之间以毫秒为单位执行带注释的方法。
   就好比 任务是1秒执行一次 但是任务执行了10秒 那么下一次任务的执行是在这个10秒结束后的1秒后
  
  long fixedDelay() default -1;

  /**
   * Execute the annotated method with a fixed period in milliseconds between the
   * end of the last invocation and the start of the next.
   *
   * @return the delay in milliseconds as a String value, e.g. a placeholder
   * @since 3.2.2
   */

  
  同上 参数为字符串的方式
  
  String fixedDelayString() default "";

  /**
   * Execute the annotated method with a fixed period in milliseconds between
   * invocations.
   *
   * @return the period in milliseconds
   */

  
   在调用之间以毫秒为单位执行带注释的方法。
   就好比 任务为1秒一次 但是1秒执行不完这个调度 但是下次任务依然会在1秒后调用
     
  long fixedRate() default -1;

  /**
   * Execute the annotated method with a fixed period in milliseconds between
   * invocations.
   *
   * @return the period in milliseconds as a String value, e.g. a placeholder
   * @since 3.2.2
   */

 
  同上 参数为字符串的方式
  
  String fixedRateString() default "";

  /**
   * Number of milliseconds to delay before the first execution of a
   * {@link #fixedRate()} or {@link #fixedDelay()} task.
   *
   * @return the initial delay in milliseconds
   * @since 3.2
   */

  
  fixedRate fixedDelay 的任务开始时 延迟多少毫秒执行
  
  long initialDelay() default -1;

  /**
   * Number of milliseconds to delay before the first execution of a
   * {@link #fixedRate()} or {@link #fixedDelay()} task.
   *
   * @return the initial delay in milliseconds as a String value, e.g. a placeholder
   * @since 3.2.2
   */

  
   同上 参数为字符串的方式
     
  String initialDelayString() default "";
}
```



**所以在选择任务执行方式的选择需要慎重避免多次执行或者不执行**

---



### Spring的Schedule调度执行方式



查阅了一下 **@EnableScheduling** 开启任务调度器注解的文档

```java
package org.springframework.scheduling.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.concurrent.Executor;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.scheduling.Trigger;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

/**
 * Enables Spring's scheduled task execution capability, similar to
 * functionality found in Spring's {@code <task:*>} XML namespace. To be used
 * on @{@link Configuration} classes as follows:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableScheduling
 * public class AppConfig {
 *
 *     // various &#064;Bean definitions
 * }</pre>
 *
 * This enables detection of @{@link Scheduled} annotations on any Spring-managed
 * bean in the container. For example, given a class {@code MyTask}
 *
 * <pre class="code">
 * package com.myco.tasks;
 *
 * public class MyTask {
 *
 *     &#064;Scheduled(fixedRate=1000)
 *     public void work() {
 *         // task execution logic
 *     }
 * }</pre>
 *
 * the following configuration would ensure that {@code MyTask.work()} is called
 * once every 1000 ms:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableScheduling
 * public class AppConfig {
 *
 *     &#064;Bean
 *     public MyTask task() {
 *         return new MyTask();
 *     }
 * }</pre>
 *
 * Alternatively, if {@code MyTask} were annotated with {@code @Component}, the
 * following configuration would ensure that its {@code @Scheduled} method is
 * invoked at the desired interval:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableScheduling
 * &#064;ComponentScan(basePackages="com.myco.tasks")
 * public class AppConfig {
 * }</pre>
 *
 * Methods annotated with {@code @Scheduled} may even be declared directly within
 * {@code @Configuration} classes:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableScheduling
 * public class AppConfig {
 *
 *     &#064;Scheduled(fixedRate=1000)
 *     public void work() {
 *         // task execution logic
 *     }
 * }</pre>
 *
 * <p>By default, will be searching for an associated scheduler definition: either
 * a unique {@link org.springframework.scheduling.TaskScheduler} bean in the context,
 * or a {@code TaskScheduler} bean named "taskScheduler" otherwise; the same lookup
 * will also be performed for a {@link java.util.concurrent.ScheduledExecutorService}
 * bean. If neither of the two is resolvable, a local single-threaded default
 * scheduler will be created and used within the registrar.
 *
 * <p>When more control is desired, a {@code @Configuration} class may implement
 * {@link SchedulingConfigurer}. This allows access to the underlying
 * {@link ScheduledTaskRegistrar} instance. For example, the following example
 * demonstrates how to customize the {@link Executor} used to execute scheduled
 * tasks:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableScheduling
 * public class AppConfig implements SchedulingConfigurer {
 *
 *     &#064;Override
 *     public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
 *         taskRegistrar.setScheduler(taskExecutor());
 *     }
 *
 *     &#064;Bean(destroyMethod="shutdown")
 *     public Executor taskExecutor() {
 *         return Executors.newScheduledThreadPool(100);
 *     }
 * }</pre>
 *
 * <p>Note in the example above the use of {@code @Bean(destroyMethod="shutdown")}.
 * This ensures that the task executor is properly shut down when the Spring
 * application context itself is closed.
 *
 * <p>Implementing {@code SchedulingConfigurer} also allows for fine-grained
 * control over task registration via the {@code ScheduledTaskRegistrar}.
 * For example, the following configures the execution of a particular bean
 * method per a custom {@code Trigger} implementation:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableScheduling
 * public class AppConfig implements SchedulingConfigurer {
 *
 *     &#064;Override
 *     public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
 *         taskRegistrar.setScheduler(taskScheduler());
 *         taskRegistrar.addTriggerTask(
 *             new Runnable() {
 *                 public void run() {
 *                     myTask().work();
 *                 }
 *             },
 *             new CustomTrigger()
 *         );
 *     }
 *
 *     &#064;Bean(destroyMethod="shutdown")
 *     public Executor taskScheduler() {
 *         return Executors.newScheduledThreadPool(42);
 *     }
 *
 *     &#064;Bean
 *     public MyTask myTask() {
 *         return new MyTask();
 *     }
 * }</pre>
 *
 * <p>For reference, the example above can be compared to the following Spring XML
 * configuration:
 *
 * <pre class="code">
 * {@code
 * <beans>
 *
 *     <task:annotation-driven scheduler="taskScheduler"/>
 *
 *     <task:scheduler id="taskScheduler" pool-size="42"/>
 *
 *     <task:scheduled-tasks scheduler="taskScheduler">
 *         <task:scheduled ref="myTask" method="work" fixed-rate="1000"/>
 *     </task:scheduled-tasks>
 *
 *     <bean id="myTask" class="com.foo.MyTask"/>
 *
 * </beans>
 * }</pre>
 *
 * The examples are equivalent save that in XML a <em>fixed-rate</em> period is used
 * instead of a custom <em>{@code Trigger}</em> implementation; this is because the
 * {@code task:} namespace {@code scheduled} cannot easily expose such support. This is
 * but one demonstration how the code-based approach allows for maximum configurability
 * through direct access to actual componentry.<p>
 *
 * @author Chris Beams
 * @author Juergen Hoeller
 * @since 3.1
 * @see Scheduled
 * @see SchedulingConfiguration
 * @see SchedulingConfigurer
 * @see ScheduledTaskRegistrar
 * @see Trigger
 * @see ScheduledAnnotationBeanPostProcessor
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {

}
```



注释中说明了调度器的使用和执行的方式。

> a local single-threaded default scheduler will be created and used within the registrar

​      

**无论有多少task，都是一个线程串行执行**



如果需要多线程去执行需要注入一个 **Executor**

正如文档中写的那样

```java
 Bean(destroyMethod="shutdown")
 public Executor taskScheduler() {
   return Executors.newScheduledThreadPool(42);
 }
```

更改任务调度线程池中线程的数量

方式有很多种

```java
  private static final ThreadFactory THREAD_FACTORY =
      new ThreadFactoryBuilder().setNameFormat("manage-thread-%s").build();
      
  @Bean
  public ScheduledExecutorService taskScheduler() {
    log.info("ScheduledExecutorService init poolSize");
    return Executors.newScheduledThreadPool(10, THREAD_FACTORY);
  }
```



```java
package mbxc.configuration;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

/**
 * Created by LXC on 2017/5/10.
 */
@Slf4j
@Configuration
public class ScheduleConfiguration implements SchedulingConfigurer {

  @Override
  public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
    ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
    taskScheduler.setPoolSize(10);
    taskScheduler.initialize();
    log.info("ThreadPoolTaskScheduler init poolSize");
    scheduledTaskRegistrar.setTaskScheduler(taskScheduler);
  }
}
```

