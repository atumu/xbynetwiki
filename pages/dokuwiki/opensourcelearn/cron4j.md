title: cron4j 

#  cron4j轻量级任务调度 
cron4j 是一个Java的任务调度框架,类似于UNIX系统下的crontab.
官网：http://www.sauronsoftware.it/projects/cron4j/index.php
Maven:
```

<dependency>
    <groupId>it.sauronsoftware.cron4j</groupId>
    <artifactId>cron4j</artifactId>
    <version>2.2.5</version>
</dependency>

```
cron4J 特点
最大的特点就是小巧,简单,功能说实话没什么可说的,就是模仿unix的crontab,门槛非常低,编程非常简单. 可以执行一些简单的定时调度功能,太复杂的还是用quartz比较好.

核心类：
  * it.sauronsoftware.cron4j.Scheduler 
  * it.sauronsoftware.cron4j.SchedulingPattern
  * it.sauronsoftware.cron4j.ProcessTask

使用步骤：
  * **Create your Scheduler instance.**
  * Schedule your actions. 任务类可以使用 **using a java.lang.Runnable or a it.sauronsoftware.cron4j.Task** instance, 时间模式可以使用**with a string or with a it.sauronsoftware.cron4j.SchedulingPattern** instance.
  * Starts the scheduler.
  * Stops the scheduler, when you don't need it anymore.

使用示例:
```

import it.sauronsoftware.cron4j.Scheduler;

public class Quickstart {

	public static void main(String[] args) {
		// Creates a Scheduler instance.
		Scheduler s = new Scheduler();
		// Schedule a once-a-minute task.每分钟执行一次
		s.schedule("* * * * *", new Runnable() {
			public void run() {
				System.out.println("Another minute ticked away...");
			}
		});
		// Starts the scheduler.
		s.start();
		// Will run for ten minutes.
		try {
			Thread.sleep(1000L * 60L * 10L);
		} catch (InterruptedException e) {
			;
		}
		// Stops the scheduler.
		s.stop();
	}

}

```
##  关于时间模式 
it.sauronsoftware.cron4j.SchedulingPattern中有说明。
同时可以通过其中的validate(java.lang.String schedulingPattern) 方法校验合法性。
Some examples:

5 * * * *
This pattern causes a task to be launched once every hour, at the begin of the fifth minute **(00:05, 01:05, 02:05 etc.).**

* * * * *
This pattern causes a task to be launched **every minute.**

* 12 * * Mon
This pattern causes a task to be launched **every minute during the 12th hour of Monday.**

* 12 16 * Mon
This pattern causes a task to be launched **every minute during the 12th hour of Monday, 16th, but only if the day is the 16th of the month.**

Every sub-pattern **can contain two or more comma separated values.**

59 11 * * 1,2,3,4,5
This pattern causes a task to be **launched at 11:59AM on Monday, Tuesday, Wednesday, Thursday and Friday.**

Values intervals are admitted and defined using the minus character.

**59 11 * * 1-5**
This pattern is equivalent to the previous one.

The slash character can be used to identify step values within a range. It can be used both in the form */c and a-b/c. The subpattern is matched every c values of the range 0,maxvalue or a-b.

*/5 * * * *
This pattern causes a task to be **launched every 5 minutes(0:00, 0:05, 0:10, 0:15 and so on).** 

3-18/5 * * * *
This pattern causes a task to be **launched every 5 minutes starting from the third minute of the hour, up to the 18th (0:03, 0:08, 0:13, 0:18, 1:03, 1:08 and so on).**

*/15 9-17 * * *
This pattern causes a task to be launched every 15 minutes between the 9th and 17th hour of the day (9:00, 9:15, 9:30, 9:45 and so on... note that the last execution will be at 17:45).

All the fresh described syntax rules can be used together.

* 12 10-16/2 * *
This pattern causes a task to be launched every minute during the 12th hour of the day, but only if the day is the 10th, the 12th, the 14th or the 16th of the month.

* 12 1-15,17,20-25 * *
This pattern causes a task to be launched every minute during the 12th hour of the day, but the day of the month must be between the 1st and the 15th, the 20th and the 25, or at least it must be the 17th.

Finally cron4j lets you combine more scheduling patterns into one, with the pipe character:

**0 5 * * *|8 10 * * *|22 17 * * ***
This pattern causes a task to be **launched every day at 05:00, 10:08 and 17:22.**

##  How to schedule a system process 
使用it.sauronsoftware.cron4j.ProcessTask可以很容易调用系统进程。
```

ProcessTask task = new ProcessTask("C:\\Windows\\System32\\notepad.exe");
Scheduler scheduler = new Scheduler();
scheduler.schedule("* * * * *", task);
scheduler.start();
// ... 

```
Arguments for the process can be supplied by using a string array instead of a single command string:
```

String[] command = { "C:\\Windows\\System32\\notepad.exe", "C:\\File.txt" };
ProcessTask task = new ProcessTask(command);
// ...

```
**Environment variables** for the process can be supplied using a second string array, whose elements have to be in the NAME=VALUE form:
```

String[] command = { "C:\\tomcat\\bin\\catalina.bat", "start" };
String[] envs = { "CATALINA_HOME=C:\\tomcat", "JAVA_HOME=C:\\jdks\\jdk5" };
ProcessTask task = new ProcessTask(command, envs);
// ...

```
The **default working directory** for the process can be changed using a third parameter in the constructor:
```

String[] command = { "C:\\tomcat\\bin\\catalina.bat", "start" };
String[] envs = { "CATALINA_HOME=C:\\tomcat", "JAVA_HOME=C:\\jdks\\jdk5" };
File directory = "C:\\MyDirectory";
ProcessTask task = new ProcessTask(command, envs, directory);
// ...

```
If you want to change the default working directory but you have not any environment variable, the envs parameter of the constructor can be set to null:
```

ProcessTask task = new ProcessTask(command, null, directory);

```
When envs is null the process inherits every environment variable of the current JVM:
Environment variables and the working directory can also be set by calling the setEnvs(String[]) and setDirectory(java.io.File) methods.
The process standard output and standard error channels can be redirected to files by using the setStdoutFile(java.io.File) and setStderrFile(java.io.File) methods:
```

ProcessTask task = new ProcessTask(command, envs, directory);
task.setStdoutFile(new File("out.txt"));
task.setStderrFile(new File("err.txt"));

```
In a siminal manner, the standard input channel can be read from an existing file, calling the setStdinFile(java.io.File) method:
```

ProcessTask task = new ProcessTask(command, envs, directory);
task.setStdinFile(new File("in.txt"));

```

##   How to schedule processes from a file 
cron4j支持类似于linu**x从一个文件加载所有的调度任务。**
通过 scheduleFile(File) 注册文件调度.
取消文件调度 descheduleFile(File) method.
Scheduled files can be retrieved by calling the ` getScheduledFiles() ` method.
Scheduled files are parsed every minute. The scheduler will launch every process declared in the file whose scheduling pattern matches the current system time.
格式如下：
0 5 * * * sol.exe
0,30 * * * * OUT:C:\ping.txt ping 10.9.43.55
0,30 4 * * * "OUT:C:\Documents and Settings\Carlo\ping.txt" ping 10.9.43.55
0 3 * * * ENV:JAVA_HOME=C:\jdks\1.4.2_15 DIR:C:\myproject OUT:C:\myproject\build.log C:\myproject\build.bat "Nightly Build"
0 4 * * * java:mypackage.MyClass#startApplication myOption1 myOption2

