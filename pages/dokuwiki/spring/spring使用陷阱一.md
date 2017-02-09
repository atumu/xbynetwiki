title: spring使用陷阱一 

#  Spring陷阱之new出的对象中的@Autowired为null 
Spring很方便，但是如果没有好好理解，一不小心我们就会掉入其陷阱。
问题描述：
```

 <!-- 配置邮件发送器 -->
  <bean id="mailSender" 
    class="org.springframework.mail.javamail.JavaMailSenderImpl"
  p:host="smtp.163.com" 
  p:port="25"
  p:username="123@163.com"
  p:password="123456" ></bean>

```

```

@Component
public class MailUtil {
	private static final Logger log=LoggerFactory.getLogger(MailUtil.class);
	@Autowired
	private JavaMailSender mailSender;
  
	private String mailto="123@outlook.com";
	private String mailFrom="123@163.com";
	private String subjectPrefix="backup data on ";
	public void sendSimpleMail(String content){
		String subject=subjectPrefix+getNowDateStr();
		SimpleMailMessage msg=new SimpleMailMessage();
		msg.setFrom(mailFrom);
		msg.setTo(mailto);
		msg.setSubject(subject);
		msg.setText(content);
          	/**运行时发生空指针异常*/
		mailSender.send(msg);  
		log.debug("邮件发送成功");
	}
	public String getNowDateStr(){
		SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		Date d=new Date();
		return sdf.format(d);
	}
}

```
```

public class DokuwikiBackService implements SchedulerService{
	@Override
	public void executeTask() {	
          	/**这个包含@Autowired的MailUtil直接被new出来了，而没有通过Spring容器创建。这就是问题所在*/
		new MailUtil().sendSimpleMail("文件备份成功"+getFilePath()+":"+etag);
				
		}
	}
		
}

```
**在运行到mailSender.send(msg); 时发生空指针异常.调试发现mailSender为null.**

分析：
**@Autowired并没有起作用**。如果是注入失败也不是提示空指针异常，而是其它异常。
我们可以看到，我这个MailUtil虽然加了个@Component注解，但是使用时我确实手动new出来一个对象，然后使用它。
经查找发现，` new出来的对象 `是直接通过JVM来创建，创建之后Spring是没有干涉的。所**以通过` new出来的对象Spring容器是没有参与权的 `。**
所以这就可以解释@Autowired没有其作用导致结果mailSender为null.

提示：
**使用@Autowired的最重要的前提是,该类必须处理Spring容器的管辖之内，而且其调用由Spring容器来负责，而不是直接被new出来。**

解决办法：
方法一、让所有包含@Autowired注解的类，**在由Spring容器管理的对象中被@Autowired注入并使用**。` 千万不要使用对包含@Autowired注解的类使用new来创建对象。 `
方法二、在MailUtil中不要使用@Autowired注解，采用

参考：
http://stackoverflow.com/questions/19896870/why-is-my-spring-autowired-field-null
http://blog.sina.com.cn/s/blog_6151984a0100oy98.html