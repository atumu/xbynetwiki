title: quartz1 

#  Quartz练习一执行cron方式调度 
```

<quartz-version>2.2.3</quartz-version>
<dependency>
				<groupId>org.quartz-scheduler</groupId>
				<artifactId>quartz</artifactId>
				<version>${quartz-version}</version>
			</dependency>
			<dependency>
				<groupId>org.quartz-scheduler</groupId>
				<artifactId>quartz-jobs</artifactId>
				<version>${quartz-version}</version>
			</dependency>

```
创建Job工作
```

import java.lang.invoke.MethodHandles;
import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.quartz.JobKey;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class SyncJob implements Job {
	private static final Logger log=LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
	private SyncMeta meta;
	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		JobKey jobKey = context.getJobDetail().getKey();
		// 获取 JobDataMap , 并从中取出参数   
        	JobDataMap data = context.getJobDetail().getJobDataMap();
        	meta=(SyncMeta) data.get("meta");
		log.info("key为{}的工作开始执行",jobKey);
		String siteMd5 = VerifySite.verifyDirForSingle(meta.getSitePath());
		meta.setSiteMd5(siteMd5);
		SyncService.sync(meta);
		log.info("key为{}的工作执行完毕",jobKey);
	}

}

```
执行调度
```

public class SiteSyncTaskScheduler {
	private static final Logger log=LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
	private static final String GROUP_NAME="site_sync_group1";
	private static final String JOB_ID="site_sync_job1";
	private static  final String TRIGGER_ID="site_sync_trigger1";
	public static Function<Void,Void> executeSiteSync(SyncMeta meta) throws SchedulerException {
		
		SchedulerFactory sf = new StdSchedulerFactory(); //创建标准调度工厂
		final Scheduler sched = sf.getScheduler(); //通过调度工厂创建调度器实例
		JobDataMap data=new JobDataMap();//创建JobDataMap,用于给Job实例传递初始化数据.
		data.put("meta", meta);
          //通过JobBuilder构建JobDetail详细描述.JobBuilder会负责创建新的Job实例,并将JobDataMap传递给Job方法参数的JobExecutionContext实例中.
		JobDetail job = JobBuilder.newJob(SyncJob.class).setJobData(data).withIdentity(JOB_ID, GROUP_NAME).build();
          
          //通过TriggerBuilder创建新的触发器,该触发器管理CronScheduleBuilder执行器,可以执行cron风格表达式方式的任务调度.如:# run every 1 hours : 0 0 */1 * * ?   
		CronTrigger trigger = TriggerBuilder.newTrigger().withIdentity(TRIGGER_ID, GROUP_NAME) 
				.withSchedule(CronScheduleBuilder.cronSchedule(meta.getCronExp())).build();
		
		log.info("工作队列开始工作,cron表达式为{}",meta.getCronExp());
          
          //用一个调度器调度-job-trigger组合的工作.
		sched.scheduleJob(job, trigger);
          //调用start方法开始调度
		sched.start();
          
          //返回一个函数接口用于绑定关闭钩子来清除调度资源-即关闭调度器
		return new Function<Void,Void>(){
			@Override
			public Void call(Void f) {
				try {
					sched.shutdown();
				} catch (SchedulerException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				return null;
			}
		};
	}
}


```

```

public class AppCleanThread extends Thread{
	@SuppressWarnings("rawtypes")
	private Function[] funcs;
	private Object[] args;
	public AppCleanThread(Function[] funcs,Object[] args){
		this.funcs=funcs;
		this.args=args;
		if(funcs==null||args==null||funcs.length!=args.length){
			throw new IllegalArgumentException("the arguments is not the thing required");
		}
	}
	@SuppressWarnings("unchecked")
	@Override
	public void run(){
		for(int i=0;i<funcs.length;i++){
			funcs[i].call(args[i]);
		}
	}
}

```

**在main函数中注册关闭钩子:**
```

		Function f1=SiteSyncTaskScheduler.executeSiteSync(meta);
		AppCleanThread clean=new AppCleanThread(new Function[]{f1},new Object[]{null});
		Runtime.getRuntime().addShutdownHook(clean);

```