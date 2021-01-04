## Quartz

基于Java实现的开源调度框架。

<img src="https://user-gold-cdn.xitu.io/2018/11/30/16763fa43e2c0909?imageslim" style="zoom:67%;" />

 trigger 和 job 是任务调度的元数据， scheduler 是实际执行调度的控制器。scheduler 由 schedulerFactory创建, 默认实现是 StdSchedulerFactory。  

Scheduler：与调度程序交互的主要API

Job：由希望由调度程序执行的组件实现的接口

JobDetail：用于定义作业的实例

Trigger：触发器，定义执行给定的作业计划的组件

Key： 将Job和Trigger注册到Scheduler时，可以为它们设置key，配置其身份属性。（withIdentify）

### 支持的调度方式：

- 按某个时间间隔无线循环

- 按每天的某个时间点，精确到秒级

- 按某一个固定时间点运行

- 通过Calendar排除某些时间

### 任务执行

任务可以是任何实现Job接口的Java类。一个任务可以有多种调度方式。

  ### Java实操

依赖：

```
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.3.0</version>
        </dependency>
```

  自定义job类重写Job的execute方法：

```
public class MyJob inplements Job {
	@Override
	public void execute(JobExecutionContext context) {
		//your job
	}
	
	public static void main(String args[]) {
		SchedulerFactory factory = new StdSchedulerFactory();
        // 从工厂里面拿到一个scheduler实例
        Scheduler scheduler = factory.getScheduler();
        String name = "name_1";
        // 真正执行的任务并不是Job接口的实例，而是用反射的方式实例化的一个JobDetail实例
        JobDetail job = JobBuilder.newJob(HelloJob.class)
        .withIdentity(name, null).build();
        // 定义一个触发器，调度策略是每隔30秒执行一次
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder
        .cronSchedule("0/30 * * * * ?").withMisfireHandlingInstructionDoNothing();
        Trigger trigger = TriggerBuilder.newTrigger()
        .withIdentity(name, null)
        .startAt(startTime)
        .withSchedule(cronScheduleBuilder).build();
        // 将任务和Trigger放入scheduler
        scheduler.scheduleJob(job, trigger);
        scheduler.start();
        // 休眠一分钟，以便任务可以执行
        TimeUnit.MINUTES.sleep(1L);
        // scheduler结束
        scheduler.shutdown(true);
	}
}
```

### cronSchedule的时间format

```
# 格式，*代表允许的全部值
* * * * * *
# 例子
每隔10秒执行一次：0/10 * * * * ?
每隔一分钟执行一次：0 0/1 * * * ?
每天8点执行一次：0 0 8 * * ?
每隔一分钟，每天8点到17点：0 0/1 8-17 * * ？
每天8点30分：0 30 8 * * ？
每周三8点30分：0 30 8 ？ * WED
每周一到周三和周六8点30分：0 30 8 ？ * MON-WED,SAT
每月最后一天8点30分：0 30 8 L * ？
每天的8点，9点30分：0 30 8,9 * * ？
```

