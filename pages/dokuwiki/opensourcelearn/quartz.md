title: quartz 

#  作业调度框架Quartz 
官网：http://quartz-scheduler.org
` Quartz `是一个开源的作业调度框架，它完全由Java写成，并设计用于J2SE和J2EE应用中。它提供了巨大的灵 活性而不牺牲简单性。**你能够用它来为执行一个作业而创建简单的或复杂的调度**。它有很多特征，如：数据库支持，集群，插件，EJB作业预构 建，JavaMail及其它，**支持cron-like表达式**等等。
本文采用Quartz 2.x版本
```

<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.2.1</version>
</dependency>

```
**任务执行**
  * 任务能够是任何实现Job接口的Java类。
  * 任务类能够被Quartz实例化,或者被你的应用框架。
  * 当一个触发器触发时，调度器会通知实例化了JobListener 和TriggerListener 接口的0个或者多个Java对象(监听器可以是简单的Java对象, EJBs, 或JMS发布者等). 在任务执行后，这些监听器也会被通知。
  * 当任务完成时，他们会返回一个JobCompletionCode ，这个代码告诉调度器任务执行成功或者失败.这个代码 也会指示调度器做一些动作-例如立即再次执行任务。
**任务持久化**
  * Quartz的设计包含JobStore接口，这个接口能被实现来为任务的存储提供不同的机制。
  * 应用JDBCJobStore, 所有被配置成“稳定”的任务和触发器能通过JDBC存储在关系数据库里。
  * 应用RAMJobStore, 所有任务和触发器能被存储在RAM里因此不必在程序重起之间保存-一个好处就是不必使用数据库。
**事务**
  * 使用JobStoreCMT（JDBCJobStore的子类），Quartz 能参与JTA事务。
  * Quartz 能管理JTA事务(开始和提交)在执行任务之间，这样，任务做的事就可以发生在JTA事务里。
**集群**
  * Fail-over.
  * Load balancing.
**监听器和插件**
  * 通过实现一个或多个监听接口，应用程序能捕捉调度事件来监控或控制任务/触发器的行为。
  * 插件机制可以给Quartz增加功能，例如保持任务执行的历史记录，或从一个定义好的文件里加载任务和触发器。
  * Quartz 装配了很多插件和监听器。
##  Quartz入门 
` quartz.properties `文件配置内容如下：
org.quartz.scheduler.instanceName: QuartzTest  
org.quartz.threadPool.threadCount: 3  
org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore  
 ```

// Grab the Scheduler instance from the Factory 
            Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
            // and start it off
            scheduler.start();
            scheduler.shutdown();

```
```

 //Scheduler instances are produced by a SchedulerFactory  
            SchedulerFactory sf = new StdSchedulerFactory();  
            Scheduler scheduler = sf.getScheduler();  
  
     
            //JobBuilder无构造函数，所以只能通过JobBuilder的静态方法newJob(Class<? extends Job> jobClass)生成JobBuilder实例  
            //withIdentity(String name,String group)参数用来定义jobKey，如果不设置，也会自动生成一个独一无二的jobKey用来区分不同的job  
            JobDetail job = JobBuilder.newJob(JobTest.class).withIdentity("job1", "group1").build();  
  
            //withIdentity(String name,String group)参数用来定义TriggerKey，如果不设置，也会自动生成一个独一无二的TriggerKey用来区分不同的trigger  
            Trigger trigger = TriggerBuilder.newTrigger().withIdentity(new TriggerKey("trigger1", "group1")).startNow()  
                            .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(2).repeatForever())  
                            .build();  

            // Tell quartz to schedule the job using our trigger  
            scheduler.scheduleJob(job, trigger);  
              
            // Start up the scheduler  
            scheduler.start();  
              
            //当前主线程睡眠2秒  
            System.out.println(Thread.currentThread().getName());  
            Thread.sleep(30*1000);  
            // shut down the scheduler  
            scheduler.shutdown(true);  

```
```

public class JobTest implements Job{  
    //Instances of Job must have a public no-argument constructor  
    public JobTest(){  
          
    }    
    public void execute(JobExecutionContext arg0) throws JobExecutionException {      
        //看打印出的当前对象每次都不一样，就等于每次执行一次任务都新建一个job实例  
        System.out.println("我的任务就是调用当前Job："+this+"不断刷屏!!!");  
    }    
}  

```
###  CronTriggerExample 
http://www.quartz-scheduler.org/documentation/quartz-2.2.x/examples/Example3.html
The program starts by getting an instance of the Scheduler. This is done by creating a StdSchedulerFactory and then using it to create a scheduler. This will create a simple, RAM-based scheduler.
```

SchedulerFactory sf = new StdSchedulerFactory();
Scheduler sched = sf.getScheduler();

```
Job #1 is scheduled to run every 20 seconds
```

JobDetail job = newJob(SimpleJob.class)
    .withIdentity("job1", "group1")
    .build();

CronTrigger trigger = newTrigger()
    .withIdentity("trigger1", "group1")
    .withSchedule(cronSchedule("0/20 * * * * ?"))
    .build();

sched.scheduleJob(job, trigger);

```
The scheduler is then started (it also would have been fine to start it before scheduling the jobs).
```

sched.start();

```
To let the program have an opportunity to run the job, we then sleep for five minutes (300 seconds). The scheduler is running in the background and should fire off several jobs during that time.
Note: Because many of the jobs have hourly and daily restrictions on them, not all of the jobs will run in this example. For example: Job #6 only runs on Weekdays while Job #7 only runs on Weekends.
```

Thread.sleep(300L * 1000L);

```
Finally, we will gracefully shutdown the scheduler:
```

sched.shutdown(true);

```
Note: passing true into the shutdown message tells the Quartz Scheduler to** wait until all jobs have completed running** before returning from the method call.
##  Spring+Quartz 
1、` Scheduler `的配置
```

<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">  
       <property name="triggers">  
           <list>  
              <ref bean="testTrigger"/>  
           </list>  
       </property>  
       <property name="autoStartup" value="true"/>  
</bean>

```  
 说明：Scheduler包含一个Trigger列表，每个Trigger表示一个作业。
2、` Trigger `的配置
```

<bean id="testTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">  
       <property name="jobDetail" ref="testJobDetail"/>  
       <property name="cronExpression" value="*/1 * * * * ?"/><!-- 每隔1秒钟触发一次 -->  
</bean>

```
3、` JobDetail `的配置
```

<bean id="testJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">   
        <property name="targetObject" ref="testJob"/>  
        <property name="targetMethod" value="execute"/>  
        <property name="concurrent" value="false"/>
        <!-- 是否允许任务并发执行。当值为false时，表示必须等到前一个线程处理完毕后才再启一个新的线程 -->  
</bean> 

``` 
4、业务类的配置
```

<bean id="testJob" class="com.cjm.web.service.quartz.TestJob"/>

```  
5、业务类源代码
```

public class TestJob {  
    public void execute(){  
        try{  
              //.......
         }catch(Exception ex){  
             ex.printStackTrace();  
         }  
     }  
} 

``` 
**说明：Spring中的Job业务类不需要继承任何父类，也不需要实现任何接口，只是一个普通的java类。**
注意：
在Spring配置和Quartz集成内容时，有两点需要注意
１、在<Beans>中不能够设置default-lazy-init="true",否则定时任务不触发，如果不明确指明default-lazy-init的值，默认是false。
２、在<Beans>中不能够设置default-autowire="byName"的属性，否则后台会报org.springframework.beans.factory.BeanCreationException错误，这样就不能通过Bean名称自动注入，必须通过明确引用注入

说明：
1）Cron表达式的格式：秒 分 时 日 月 周 年(可选)。
![](/data/dokuwiki/opensourcelearn/pasted/20160602-160431.png)
特殊字符
  * * 表示所有值 ；
  * ? 表示未说明的值，即不关心它为何值；
  * - 表示一个指定的范围；
  * , 表示附加一个可能值；
  * / 符号前表示开始时间，符号后表示每次递增的值；
  * 
  * L ("last") "L" 用在day-of-month字段意思是 "这个月最后一天"；用在 day-of-week字段, 它简单意思是 "7" or "SAT"。 如果在day-of-week字段里和数字联合使用，它的意思就是 "这个月的最后一个星期几" – 例如： "6L" means "这个月的最后一个星期五". 当我们用“L”时，不指明一个列表值或者范围是很重要的，不然的话，我们会得到一些意想不到的结果。
  * 
  * W ("weekday") –只能用在day-of-month字段。用来描叙最接近指定天的工作日（周一到周五）。例如：在day-of-month字段用“15W”指“最接近这个月第15天的工作日”，即如果这个月第15天是周六，那么触发器将会在这个月第14天即周五触发；如果这个月第15天是周日，那么触发器将会在这个月第16天即周一触发；如果这个月第15天是周二，那么就在触发器这天触发。注意一点：这个用法只会在当前月计算值，不会越过当前月。“W”字符仅能在day-of-month指明一天，不能是一个范围或列表。
  * 也可以用“LW”来指定这个月的最后一个工作日。
  * 
  * # -只能用在day-of-week字段。用来指定这个月的第几个周几。例：在day-of-week字段用"6#3"指这个月第3个周五（6指周五，3指第3个）。如果指定的日期不存在，触发器就不会触发。
  * 
  * C ("calendar") – 指和calendar联系后计算过的值。例：在day-of-month 字段用“5C”指在这个月第5天或之后包括calendar的第一天；在day-of-week字段用“1C”指在这周日或之后包括calendar的第一天。
2）Cron表达式范例：
每隔5秒执行一次：*/5 * * * * ?
每隔1分钟执行一次：0 */1 * * * ?
每天23点执行一次：0 0 23 * * ?
每天凌晨1点执行一次：0 0 1 * * ?
每月1号凌晨1点执行一次：0 0 1 1 * ?
每月最后一天23点执行一次：0 0 23 L * ?
每周星期天凌晨1点实行一次：0 0 1 ? * L
在26分、29分、33分执行一次：0 26,29,33 * * * ?
每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?
0 0 12 * * ?
每天中午12点
0 15 10 * * ? 2005
在2005年的每天10：25
0 10,44 14 ? 3 WED
在3月里每个周三的14：10和14：44
0 15 10 ? * 6L 2002-2005
从2002年到2005年里，每个月的最后一个星期五的10：15
0 0 12 1/5 * ?
从当月的第一天开始，然后在每个月每隔5天的12：00
0 15 10 ? * 6#3
每个月第3个周五的10：15                 
更多详细内容建议参考：http://blog.csdn.net/bubei/article/details/2108778

参考：http://luan.iteye.com/blog/1629148
http://www.oschina.net/question/8676_9032