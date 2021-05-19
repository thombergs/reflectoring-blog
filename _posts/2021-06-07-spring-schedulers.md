---
title: "Scheduling with Spring Boot"
categories: [spring-boot]
date: 2021-03-05 00:00:00 +0530
modified: 2021-03-05 00:00:00 +0530
author: default
excerpt: "In this article, we're going to look at spring schedulers, different annotations for scheduling tasks, how it works internally, and how to schedule tasks without an annotation."
image:
  auto: 0061-cloud
---

We often come across use cases where we need to execute some task periodically. It could be at a fixed time in the day or maybe every few hours. One way to do it in an old-fashioned way  would be to expose endpoints for those tasks and calling those endpoints from an external scheduler. But what if I told you we do not need to do this? The Spring developers already took care of this and in this article we'll be discussing Spring schedulers.

Some cases where we can use Spring schedulers are:

* calling an API periodically, 
* reading a file from an SFTP location which is uploaded there once a day,
* or maybe checking your mail to see if your manager is on leave every morning :). 
  
Anything that you want to do periodically without depending on any external System can be done by Spring schedulers

## Adding Spring Scheduler to a Project

We don't need to add any additional dependency when working with a Spring Boot application, as it comes with the standard Spring Boot starter.

## Important Annotations for Spring Schedulers

We need to add `@EnableScheduling` to one of our `@Configuration` classes to enable detection of `@Scheduled` annotations. Often, we use it on top of our main application class, but no one is stopping us to use it on of our other `@Configuration` classes:

```java
@SpringBootApplication
@EnableScheduling
public class SpringSchedulerApplication {
  public static void main(String[] args) {
    SpringApplication.run(SpringSchedulerApplication.class, args);
  }
}
```

Once the scheduling is enabled, we can use the `@Scheduled` annotation on methods that define the task we want to schedule. More about the `@Scheduled` annotation in the next sections.
  

## What are Spring Scheduler Tasks

Strictly talking about syntax, **any void method which does take any parameters and is decorated with @Scheduled annotation** can be termed as a spring scheduler task. 


## Scheduling Tasks with `@Scheduled`


As of now, we know that `@Scheduled` is the way to schedule tasks, but we haven't seen it in action. Let's look at different ways on how to use this annotation:

### Schedule tasks with `FixedRate`
Using this would execute our task, after every n milliseconds, which you provided. **An important thing to know here is that every execution is independent i.e. it will execute our task `EVEN if the previous execution is still in progress`**.
```java
@Scheduled(fixedRate = 1000)
public void callApiViaFixedRate() {
  System.out.println("calling API using fixed rate");
}
```
### Schedule tasks with `FixedDelay`
Using this would ensure, the gap between the execution of our task is always n milliseconds, i.e. **our task will invoke `ONLY after the previous execution is complete.`**
```java
@Scheduled(fixedDelay = 1000)
public void callApiViaFixedDelay() {
 System.out.println("calling API using fixed delay");
}
```
### Schedule tasks with `Cron Expression`
Generally, our use cases are to execute tasks daily at some fixed time or weekly or any other `calendar schedule`, for those purposes we use cron expression.
```java
@Scheduled(cron = "0/30 * * * * ?")
public void callApiViaCron() {
 System.out.println("calling API using cron expression");
}
```

### Support for `Initial Delay's and Time Zone` 
For Fixed Rate, Fixed Delay and Cron, we can add **initial delays** for the FIRST execution-only, that is our task will be executed `after the initial delay specified`. So we can do some preparatory things if we want to.
```java
@Scheduled(fixedRate = 1000, initialDelay = 200)
public void callApiViaFixedRateWithInitialDelay() {
 System.out.println("calling API using fixed rate with initial delay");}
```   

For the Cron task, we can also specify **zones**. For example, if NASA wants to stop the Perseverance rover’s heating system at 9 AM Martian time, they can use a spring scheduler:). The only problem is to add Martian TimeZone Support, rest is already covered.

```java
@Scheduled(cron = "0 0 9 * * ?", zone = "Milky Way/Mars")
public void callApiViaCronForASpecificZone() {
 System.out.println("Stop heating systems, its warm enough");
}   
```  

## Configuring a Custom Thread Pool
**All tasks are queued in the same thread pool that's reserved for scheduled tasks.**

We can define a custom thread pool and our scheduled tasks will run over it. 

```java
@SpringBootApplication
@EnableScheduling
public class SpringSchedulerApplication {
 public static void main(String[] args) {
   SpringApplication.run(SpringSchedulerApplication.class, args);
 }

//custom thread pool size of 10, define it as per need
@Bean(destroyMethod = "shutdown")
public ThreadPoolTaskScheduler taskScheduler() {
 ThreadPoolTaskScheduler taskScheduler = new     ThreadPoolTaskScheduler();
  taskScheduler.setPoolSize(10);    
  return taskScheduler;
 }
}
```

## The Internals of Spring's Scheduling Framework


Let's do a little deep dive to understand how it all works.

The Spring Framework provides abstractions for scheduling tasks with the `TaskScheduler` interface, and the annotations we discussed earlier are all annotation support for these methods below:
      

```java
public interface TaskScheduler {

  //remember there is not annotation support for this.
  //we will cover this later in this article
  ScheduledFuture schedule(Runnable task,
   Trigger trigger);


  ScheduledFuture schedule(Runnable task, 
    Date startTime);

  ScheduledFuture scheduleAtFixedRate(Runnable task,
   Date startTime, long period);

  ScheduledFuture scheduleAtFixedRate(Runnable task,
   long period);

  ScheduledFuture scheduleWithFixedDelay(Runnable task,
   Date startTime, long delay);

  ScheduledFuture scheduleWithFixedDelay(Runnable task, 
    long delay);

}
```

Every task is wrapped into a `Task` object, which contains the call to our `@Scheduled` method as a `Runnable`:

```java
public class Task {
  private final Runnable runnable;

  public Task(Runnable runnable) {
    Assert.notNull(runnable, "Runnable must not be null");
    this.runnable = runnable;
  }

  public Runnable getRunnable() {
    return this.runnable;
  }
}
```

A `Task`, in turn, is wrapped by a `ScheduledTask` object:

```java
//this is protected, so you cannot  create an instance of this manually 
public final class ScheduledTask {
 private final Task task;
 @Nullable
 volatile ScheduledFuture<?> future;

 ScheduledTask(Task task) {
   this.task = task;
 }
 public void cancel() {
   ScheduledFuture<?> future = this.future;
   if (future != null) {
     future.cancel(true);
   }
 }
}


```

Knowing the basic classes, we can now look at where all the magic happens.

`ScheduledAnnotationBeanPostProcessor` is the class that holds the responsibility of creating a `ScheduledTask` for each `@Scheduled` annotation it finds.

This class registers a task based on whatever flavour we have used to define it:

* if we used `@Scheduled(cron="...")`, it will register a `CronTask`
* if we used `@Scheduled(fixedRate="")`, it will register a `FixedRateTask`
* ...

We won’t look at all the task types, you can look at the [processScheduled()](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/scheduling/annotation/ScheduledAnnotationBeanPostProcessor.java#L391) method for more detail.

Let's look at the `CronTask` class, for example: 

```java
public class CronTask extends TriggerTask {
 private final String expression;

 public CronTask(Runnable runnable, String expression) {
   this(runnable, new CronTrigger(expression));
 }

 public CronTask(Runnable runnable, CronTrigger cronTrigger) {
   super(runnable, cronTrigger);
   this.expression = cronTrigger.getExpression();
 }

 public String getExpression() {
   return this.expression;
 }
}
``` 

The cron expression we define in the annotation (`@Scheduled(cron="...")`) is now stored in the field `expression` of this class. The expression is final, which means, we can't update it while the application is running. If we want to change it, we will need to register a new task and stop this one.

The same goes for tasks of any other flavor. **All tasks are immutable when we create them with the `@Scheduled` annotation**.

If we want to have mutable tasks, we will have create them manually, which we will look at in the next section.

## Manually Scheduling (Mutable) Tasks

The first question that comes to any developer's mind is **Why schedule a task without annotation?**        
There are two primary reasons for it:
 * We want a custom trigger logic for our task.
 * We want a mutable task, i.e. ability to change the task's configuration at run time.

In the next section, we will see both of these reasons in a little more depth.


### Changing a task's configuration at runtime
As we learned in the previous section, tasks created with the `@Scheduled` annotation tasks are immutable. For example, if we scheduled a task that pulls data at 4 AM every day and we want to change the time, we can’t. We will need to stop the application and update the cron expression.

For example, maybe we can use [spring-cloud-config](https://spring.io/projects/spring-cloud-config) to update task configurations, without having downtime for our application) or may we can update cron via rest controller.

In this article we will see spring-cloud-config variation, as it can have more use cases in industry. 

To implement these requirements, we need to create a custom `Trigger` implementation, which uses RefreshScope:

```java
@RefreshScope// refresh bean when actuator refresh is done for reflecting cloud config changes, without restarting the app.
@Component
public class CloudConfigTrigger implements Trigger {

  // Cron expression from cloud config
  @Value("${reflectoring.cron}")
  private String cron;

  @Override
  public Date nextExecutionTime(TriggerContext triggerContext) {
    CronTrigger crontrigger = new CronTrigger(cron);
    Date nextRunDate = crontrigger.nextExecutionTime(triggerContext);
    return nextRunDate;
  }
}
```
### Custom trigger logic
we can have use cases, where we want to run tasks based on some custom logic. **Let's say we want to calculate the next execution time based on that day's weather forecast.**


To implement these requirements, we need to create a custom `Trigger` implementation, which can hold our custom logic:

```java
@Component
public class WeatherForecastTrigger implements Trigger {

  //custom trigger logic
  private String cronAffectedByWeather() {
    int currentTemperature = getCurrentTemperature();

    //if temperature is less then 25 degrees, execute task every 10 seconds 
    //from next scheduled time
    if (currentTemperature < 25) {
      return "0/10 * * * * ?";
    }
    //if temperature is greater then 25 degrees, execute task every 30 seconds
    // from next scheduled time
    return "0/30 * * * * ?";

  }

  // This gives us the current temperature between the range 0 and 50
  private int getCurrentTemperature() {
    return (int) Math.random() * (50);
  }

  @Override
  public Date nextExecutionTime(TriggerContext triggerContext) {
    CronTrigger crontrigger = new CronTrigger(cronAffectedByWeather());
    Date nextRunDate = crontrigger.nextExecutionTime(triggerContext);
    return nextRunDate;
  }

}
```

To register our task with Spring's scheduling framework, we need to implement the `SchedulingConfigurer` interface:

```java
package refactoring.io.springscheduler.service;

import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.stereotype.Component;

import java.util.concurrent.ScheduledFuture;

// We will register our custom trigger tasks
@Component
public class SchedulerConfig implements SchedulingConfigurer {

  private final WeatherForecastTrigger weatherForecastTrigger;

  private final CloudConfigTrigger cloudConfigTrigger;

  private ScheduledFuture<?> weatherTaskFuture;
  private ScheduledFuture<?> cloudConfigTaskFuture;

  public SchedulerConfig(WeatherForecastTrigger weatherForecastTrigger,
                         CloudConfigTrigger cloudConfigTrigger) {
    this.weatherForecastTrigger = weatherForecastTrigger;
    this.cloudConfigTrigger = cloudConfigTrigger;
  }

  @Override
  public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {

    TaskScheduler taskScheduler = taskRegistrar.getScheduler();
    registerWeatherForecastTask(taskScheduler);
    registerCloudConfigTask(taskScheduler);
  }

  //register weather forecast task
  private void registerWeatherForecastTask(TaskScheduler taskScheduler) {

    weatherTaskFuture = taskScheduler.schedule(
        () -> {
          System.out.println("call API from weather forecast task");
        }, weatherForecastTrigger);
  }

  //register spring-cloud task
  private void registerCloudConfigTask(TaskScheduler taskScheduler) {
    cloudConfigTaskFuture = taskScheduler.schedule(
        () -> {
          System.out.println("call API from cloud sconfig task");
        }, cloudConfigTrigger);
  }

}
``` 


Now once you start the app, we will see two tasks running, one WeatherForecast task will be alternating between 2 crons shown in the above example based on the value from getCurrentTemperature() function, and second, the CloudConfig task will start with the initial value in your cloud-config and once you change the value in cloud-config and hit actuator refresh. CloudConfig task will start running on updated taken cron from the cloud-config

## Conclusion

So In this article, we have seen how to use @Scheduled annotation, different flavors of @Scheduled annotation, an overview of its internal working, and how to schedule a mutable task without any annotations With all of this information. I am sure you will get the motivation to deep dive in spring internals and implement any custom solution required in your projects.



