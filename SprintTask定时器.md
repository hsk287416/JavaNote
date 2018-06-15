# 1. Spring Task简介
Spring Task主要是用来实现定时任务。

目前，Java有三种方法能够实现定时任务。

**第一种 java.util.Timer类&java.util.TimerTask类**

但是这种方法，只能通过指定时间间隔来定时，而不能直接指定时间。

**第二种 Quartz框架**

Quartz框架的功能比较强大，它既能指定时间间隔，也能直接指定时间。但是使用起来比较复杂。

**第三种 Spring框架**

Spring从3.0版本开始自带task调度工具，它能实现Quartz所有的功能，并且使用起来很简单。

# 2. 纯XML配置方式实现Spring Task

## 2.1 简单定时任务
**实现功能：** 每隔一定时间就会执行某个任务。

首先，我们写一个任务（打印字符串）：

***TaskService.java***
```java
public interface TaskService {
    /**
     * 第一个定时任务
     */
    void firstTask();

    /**
     * 第二个定时任务
     */
    void secondTask();
}
```

***TaskServiceImpl.java***
```java
public class TaskServiceImpl implements TaskService {
    @Override
    public void firstTask() {
        System.out.println(new Date() + "这是第1个定时任务");
    }

    @Override
    public void secondTask() {
        System.out.println(new Date() + "这是第2个定时任务");
    }
}
```

然后需要配置 ***applicationContext.xml*** 文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd">

    <!--任务类-->
    <bean id="taskService" class="com.example.demo.service.TaskServiceImpl"></bean>

    <!--配置定时规则-->
    <task:scheduled-tasks>
        <!--initial-delay: 服务器（Tomcat）运行之后等多少毫秒开始执行定时任务-->
        <!--fixed-delay：每个多少毫秒执行一次任务-->
        <task:scheduled ref="taskService" method="firstTask" initial-delay="1000" fixed-delay="1000"/>
        <task:scheduled ref="taskService" method="secondTask" initial-delay="2000" fixed-delay="3000"/>
    </task:scheduled-tasks>
</beans>
```

> 注意：Spring Boot环境下，需要在main方法所在文件的类名上添加这样一段注解：

```java
@ImportResource("classpath:applicationContext.xml")
```

## 2.2 复杂定时任务

### 2.2.1 cron表达式

cron表达式是一个字符串，用来定义复杂的定时规则。它由7个部分构成，每个部分中间用空格隔开，每个部分的含义如下：

| 组成部分 | 含义 | 取值范围 |
| -- | -- | -- |
| 第1部分 | Seconds(秒) | 0-59 |
| 第2部分 | Minutes(分) | 0-59 |
| 第3部分 | Hours(时) | 0-23 |
| 第4部分 | Day-of-Month(天) | 1-31 |
| 第5部分 | Month(月) | 0-11 |
| 第6部分 | Day-of-Week(星期几) | 1-7(1表示星期日) |
| 第7部分 | Year(年) *可选* | 1970-2099 |

另外，cron表达式还可以包含一些特殊符号来定义更加灵活的定时规则，如下表所示：

| 符号 | 含义 |
| -- | -- |
| ？ | 表示不确定的值，任意的一天 |
| * | 表示整个时间段 |
| , | 设置多个值，例如“26,29,33”，表示在26分，29分和33分的时候分别执行一次任务 |
| - | 设置取值范围，例如“5-20”，表示从5分到20分，每分钟执行一次任务 |
| / | 设置频率或间隔，例如“1/15”，表示从1分开始，每隔15分钟执行一次任务 |
| L | 用于每月或每周，表示每月的最后一天，或每个月的最后一个星期几，例如“6L”，表示每个月的最后一个星期五 |
| W | 表示离给定日期最近的工作日，例如“15W”放在每月（day-of-month）上，表示离本月15日最近的工作日 |
| # | 表示该月第几个星期几，例如“6#3”，表示该月第3个星期五 |

**cron表达式示例：**

| cron表达式 | 含义 |
| -- | -- |
| */5 * * * * ? | 每隔5秒执行一次任务 |
| 0 0 23 * * ? | 每天23点执行一次任务 |
| 0 0 1 1 * ? | 每月1号凌晨1点执行一次任务 |
| 0 0 23 L * ? | 每月最后一天23点执行一次任务 |
| 0 26,29,33 * * * ? | 在每个小时的26分、29分、33分时各执行一次任务 |
| 0 0 0,13,18,21 * * ? | 每天的0点、13点、18点、21点各执行一次任务 |
| 0 0/30 9-17 * * ? | 朝九晚五工作时间内每半个小时执行一次任务 |
| 0 15 10 ? * 6#3 | 每月的第三个星期五上午10:15时执行一次任务 |

### 2.2.2 复杂定时任务配置

***applicationContext.xml*** 文件如下配置即可：

```xml
<!--任务类-->
<bean id="taskService" class="com.example.demo.service.TaskServiceImpl"></bean>

<!--每个2秒执行一次-->
<task:scheduled-tasks>
    <task:scheduled ref="taskService" method="firstTask" cron="*/2 * * * * ?"/>
</task:scheduled-tasks>
```

# 3. 全注解方式实现Spring Task

在SpringBoot中，我们一般是不会使用applicationContext.xml文件的。

因此，我们更常使用全注解方式实现Spring Task。做法如下：

首先需要在main方法所在class文件的上加入以下注解：

```java
@SpringBootApplication
@EnableScheduling   //启用定时任务
public class Demo01Application {

	public static void main(String[] args) {
		SpringApplication.run(Demo01Application.class, args);
	}
}
```

然后这样来定义定时任务即可：
```java
@Service
public class TaskServiceImpl implements TaskService {

    @Scheduled(initialDelay = 1000, fixedDelay = 1000)
    @Override
    public void firstTask() {
        System.out.println(new Date() + "这是第1个定时任务");
    }

    @Scheduled(cron = "*/5 * * * * ?")
    @Override
    public void secondTask() {
        System.out.println(new Date() + "这是第2个定时任务");
    }
}
```
