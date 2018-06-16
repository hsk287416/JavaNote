# 1. 环境准备

我们将以Spring Boot作为项目环境，并导入Quatz Schedule的maven配置：
```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-quartz -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

# 2. 入门实例

首先我们先准备一个任务类。任务类一定要是*QuartzJobBean*的子类，并且实现了*executeInternal*方法：

***WeatherDataSyncJob.java***
```java
@Slf4j
public class WeatherDataSyncJob extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        log.info("我来获取天气信息");
    }
}
```

然后我们需要准备一个注解类，注解类中需要生成两个Bean，一个是JobDetail，另一个是Trigger：
***QuartzConfiguration.java***
```java
@Configuration
public class QuartzConfiguration {

    @Bean
    public JobDetail weatherDataSyncJobDetail() {
        // JobBuilder.newJob()用来创建一个Job
        // JobBuilder.newJob()中的参数是要执行的任务的类
        // withIdentity()方法可以为Job指定一个名称
        return JobBuilder.newJob(WeatherDataSyncJob.class)
                .withIdentity("weather-data-sync-job")
                .storeDurably()
                .build();
    }

    @Bean
    public Trigger weatherDataSyncTrigger() {
        SimpleScheduleBuilder simpleScheduleBuilder = SimpleScheduleBuilder
                .simpleSchedule()
                .withIntervalInSeconds(5)   //每5秒执行一次
                .repeatForever();           //永远执行

        return TriggerBuilder
                .newTrigger()
                .forJob(weatherDataSyncJobDetail())
                .withIdentity("weather-data-sync-trigger")
                .withSchedule(simpleScheduleBuilder)
                .build();
    }
}
```