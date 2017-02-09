title: slf4j与log4j_commonslogging 

#  slf4j与log4j、commons logging 

#  Log4j使用总结 
##  基本使用： 

配置
代码中使用
日志文件输出路径配置方式。
补充知识点
配置文件总结示例

Log4j由三个重要的组件构成：日志信息的优先级，日志信息的输出目的地，日志信息的输出格式。
日志信息的优先级从高到低有ERROR、WARN、INFO、DEBUG，分别用来指定这条日志信息的重要程度；
日志信息的输出目的地指定了日志将打印到控制台还是文件或数据库或email或其他等中；
而输出格式则控制了日志信息的显示内容。
1.定义配置文件：（支持xml或properties格式、）

1.1配置根Logger，其语法为：
log4j.rootLogger = [ level ] , appenderName, appenderName, …
其中，level 是日志记录的优先级，分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者您定义的级别。
<wrap hi>Log4j建议只使用四个级别，优先级从高到低分别是ERROR、WARN、INFO、DEBUG。
通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关。比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。
appenderName就是指定日志信息输出到哪个地方。您可以同时指定多个输出目的地。</wrap>
1.2.配置日志信息输出目的地Appender，其语法为
log4j.appender.appenderName = fully.qualified.name.of.appender.class
log4j.appender.appenderName.option1 = value1
…
log4j.appender.appenderName.option = valueN
其中，Log4j提供的appender有以下几种：
org.apache.log4j.ConsoleAppender（控制台），
org.apache.log4j.FileAppender（文件），
org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），
<wrap hi>org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件并可支持保存文件索引的最大数及最多的日志文件数。），
org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）</wrap>

1.3配置日志信息的格式（布局），其语法为：
log4j.appender.appenderName.layout = fully.qualified.name.of.layout.class
log4j.appender.appenderName.layout.option1 = value1
…
log4j.appender.appenderName.layout.option = valueN
其中，Log4j提供的layout有以下几种：
org.apache.log4j.HTMLLayout（以HTML表格形式布局），
org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），
org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

2、在代码中使用log4j
2.1:读取配置文件
当获得了日志记录器之后，第二步将配置Log4j环境，其语法为：
BasicConfigurator.configure ()： 自动快速地使用缺省Log4j环境。log4j.properties
PropertyConfigurator.configure ( String configFilename) ：读取使用Java的特性文件编写的配置文件。
DOMConfigurator.configure ( String filename ) ：读取XML形式的配置文件。
2.2.得到记录器
使用Log4j，第一步就是获取日志记录器，这个记录器将负责控制日志信息。其语法为：
public static Logger getLogger( String name)，
通过指定的名字获得记录器，如果必要的话，则为这个名字创建一个新的记录器。Name一般取本类的名字，比如：
static Logger logger = Logger.getLogger ( ServerWithLog4j.class.getName () ) ;
2.3插入记录信息（格式化日志信息）
当上两个必要步骤执行完毕，您就可以轻松地使用不同优先级别的日志记录语句插入到您想记录日志的任何地方，其语法如下：
Logger.debug ( Object message ) ;
Logger.info ( Object message ) ;
Logger.warn ( Object message ) ;
Logger.error ( Object message ) ;

##  2.4在Servlet中的使用： 

配置一个单独的Servlet用于随着webapp的初始化初始化日志配置：

<servlet>
   <servlet-name>log4j-init</servlet-name>
   <servlet-class>com.foo.Log4jInit</servlet-class>
   <init-param>
     <param-name>log4j-init-file</param-name>
     <param-value>WEB-INF/classes/log4j.lcf</param-value>
   </init-param>
   <load-on-startup>1</load-on-startup>
 </servlet>


public class Log4jInit extends HttpServlet {
 public
 void init() {
   String prefix =  getServletContext().getRealPath("/");
   String file = getInitParameter("log4j-init-file");
   // if the log4j-init-file is not set, then no point in trying
   if(file != null) {
     PropertyConfigurator.configure(prefix+file);
   }
 }
` 注意：以上使用部分仅作为了解，一般在实际项目中很少直接使用Log4j的API来操作，我们一般通过日志系统抽象层来面向接口一致地编程，例如SLF4J，commons logging. `

##  关于设置文件路径的补充： 

设置日志文件的路径大体有以下三种方式：
1.使用既定环境遍历如tomcat的${catalina.home}:
log4j.appender.R.File=${CATALINA_HOME}/example.log不过这种方式不便于移植，
2.覆盖RollingFileAppender的setFile方法，麻烦。
3.在加载log4j.properties文件之前配置Systemt.setProperty("log_path",log_path);推荐
高优先级、自启动的Servlet里(比如前面讲过的Log4jInitServlet)获得应用的发布目录，这是很容易得到的，
比如String home = servletContext.getRealPath("/");然后调用System.setPropertity("web_home",home),“web_home”名字可以随意。
则log4j.properties里的配置为：log4j.appender.R.File=${web_home}/logs/log.log.则日志会输出到web应用发布目录里的logs目录里。
<servlet>
<servlet-name>log4j-init</servlet-name>
<servlet-class>Log4jInit</servlet-class>
<init-param>
<param-name>log_path</param-name>
<param-value>logs</param-value>避免硬编码确定日志文件父目录。
</init-param>
        <load-on-startup>0</load-on-startup>
</servlet>
public class Init extends HttpServlet {
public void init(ServletConfig config) {
     String path=config.getServletContext().getRealPath("/");
    String path=path+config.getInitParameter("log_path");
    System.setProperty("log_path",path);
  }
}
log4j.properties文件中的配置为：
log4j.appender.R.File=${log_path}/log.log.

4.动态确定日志文件位置，这样需要通过编码操作RollingFileAppender。除非必须这么做，一般我们没有太大必要。
static{
       Appender appender = logger.getAppender("RING");
       if(appender instanceof FileAppender){
           FileAppender fappender = (FileAppender)appender;
           if(!fappender.getFile().startsWith("App1")){
               fappender.setFile("App1"+fappender.getFile());
               fappender.activateOptions();
           }
       }
通过调用appender的activateOptions方法来激活针对appender的修改，之后的日志就记录在新的日志文件中了。
###  3. 关于RollingFileAppender:的几点补充： 

举例：

log4j.rootLogger=debug, stdout, R
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
# Pattern to output the caller's file name and line number.
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] (%F:%L) - %m%n
log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=example.log
log4j.appender.R.MaxFileSize=100KB #指定单个日志文件最大大小，超出这个大小就会Rolling为example.log.1
# Keep one backup file
log4j.appender.R.MaxBackupIndex=1//指定最大日志索引example.log.1
log4j.appender.R.Append=true //true:添加 false:覆盖  
#log4j.appender.R.MaxFileSize=10KB //文件最大尺寸  
#log4j.appender.R.MaxBackupIndex=1 //备份数  

log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n


###  关于PatternLayout的几点补充 


格式
%m 输出代码中指定的消息
%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL
%r 输出自应用启动到输出该log信息耗费的毫秒数
%c 输出所属的类目，通常就是所在类的全名
%t 输出产生该日志事件的线程名
%n 输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"
%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
%l 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(Test Log4.java:10) 

%x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中

可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式


一般比较常用的是：%p,%m,%t,%d,%l,%n，%x

例如log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %-5p ] - %-5l- %-5x - %-5m%n
关于FileAppender的补充：

log4j.appender.FILE.Append=true//是否追加


###  4. 关于其他输出目的地的说明示例： 


log4j.rootLogger=DEBUG,CONSOLE,A1,im  
# 应用于控制台  
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender  
log4j.appender.Threshold=DEBUG  
log4j.appender.CONSOLE.Target=System.out  
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout  
log4j.appender.CONSOLE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
#log4j.appender.CONSOLE.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[thread] n%c[CATEGORY]%n%m[MESSAGE]%n%n  
#应用于文件  
log4j.appender.FILE=org.apache.log4j.FileAppender  
log4j.appender.FILE.File=file.log  
log4j.appender.FILE.Append=false  
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout  
log4j.appender.FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
# 应用于文件回滚  
log4j.appender.ROLLING_FILE=org.apache.log4j.RollingFileAppender  
log4j.appender.ROLLING_FILE.Threshold=ERROR  
log4j.appender.ROLLING_FILE.File=rolling.log //文件位置,也可以用变量${java.home}、rolling.log  
log4j.appender.ROLLING_FILE.Append=true //true:添加 false:覆盖  
log4j.appender.ROLLING_FILE.MaxFileSize=10KB //文件最大尺寸  
log4j.appender.ROLLING_FILE.MaxBackupIndex=1 //备份数  
log4j.appender.ROLLING_FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.ROLLING_FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
#应用于socket  
log4j.appender.SOCKET=org.apache.log4j.RollingFileAppender  
log4j.appender.SOCKET.RemoteHost=localhost  
log4j.appender.SOCKET.Port=5001  
log4j.appender.SOCKET.LocationInfo=true  
# Set up for Log Facter 5  
log4j.appender.SOCKET.layout=org.apache.log4j.PatternLayout  
log4j.appender.SOCET.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[thread]%n%c[CATEGORY]%n%m[MESSAGE]%n%n  
 
# Log Factor 5 Appender  
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender  
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000  
# 发送日志给邮件  
log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender  
log4j.appender.MAIL.Threshold=FATAL  
log4j.appender.MAIL.BufferSize=10  
log4j.appender.MAIL.From=web@www.wuset.com  
log4j.appender.MAIL.SMTPHost=www.wusetu.com  
log4j.appender.MAIL.Subject=Log4J Message  
log4j.appender.MAIL.To=web@www.wusetu.com  
log4j.appender.MAIL.layout=org.apache.log4j.PatternLayout  
log4j.appender.MAIL.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
# 用于数据库  
log4j.appender.DATABASE=org.apache.log4j.jdbc.JDBCAppender  
log4j.appender.DATABASE.URL=jdbc:mysql://localhost:3306/test  
log4j.appender.DATABASE.driver=com.mysql.jdbc.Driver  
log4j.appender.DATABASE.user=root  
log4j.appender.DATABASE.password=  
log4j.appender.DATABASE.sql=INSERT INTO LOG4J (Message) VALUES (’[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n’)  
log4j.appender.DATABASE.layout=org.apache.log4j.PatternLayout  
log4j.appender.DATABASE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
#自定义Appender  
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender  
log4j.appender.im.host = mail.cybercorlin.net  
log4j.appender.im.username = username  
log4j.appender.im.password = password  
log4j.appender.im.recipient = corlin@cybercorlin.net  
log4j.appender.im.layout=org.apache.log4j.PatternLayout  
log4j.appender.im.layout.ConversionPattern =[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  



参考：https://www.ibm.com/developerworks/cn/java/l-log4j/

http://www.cnblogs.com/licheng/archive/2008/08/23/1274566.html
http://kdboy.iteye.com/blog/208851

#  apache commons Logging与Log4j结合的使用 
apache commons Logging 是一种JCL ,java commons Logging.
它的宗旨在于取消各种日志框架实现之间的差异，以统一的一致地接口API来操作日志。从而实现高度的日志系统抽象，你无需从新编码就可以切换不同的日志框架实现。所以所commons Loging屏蔽了具体日志实现了之间的差异。（思路上类似于JDBC的抽象）。我们只需要面对Commons Logging操作日志即可。
其实与commons Logging体系类似的还有SLF4J，SLF4J是log4j的作者另起灶炉搞的，除了这个抽象体系外，还有一个实现用以取代log4j，就是logback,。 SLF4J也可以兼容各种日志框架实现。但是它的API更为简单，操作更为方便，更强大。SLF4J+Log4j目前很流行，SLF4J+Logback是SLF4j官方推荐的，据说比commonsLogging+log4j快几倍。同时也更方便。咱不谈它。

最常见的commons Logging实现是Log4j.所以我们主要介绍commons Loggins 与log4j的结合使用。
那commons logging如何找到或确定使用哪个实现呢。以下为其与log4j整合的大致思路：

1)        首先在classpath下寻找自己的配置文件commons-logging.properties，如果找到，则使用其中定义的Log实现类；
如org.apache.commons.logging.Log = org.apache.commons.logging.impl.SimpleLog
2)        如果找不到commons-logging.properties文件，则在查找是否已定义系统环境变量org.apache.commons.logging.Log，找到则使用其定义的Log实现类；如：
System.getProperties().setProperty(LogFactory.class.getName(),
                                   Log4jFactory.class.getName());
3)        否则，查看classpath中是否有Log4j的包，如果发现，则自动使用Log4j作为日志实现类；
4)        否则，使用JDK自身的日志实现类（JDK1.4以后才有日志实现类）；
5)        否则，使用commons-logging自己提供的一个简单的日志实现类SimpleLog；

可见我们只要我们将log4j的jar文件放在classpath中，就无需配置，就可以直接使用commons logging与log4j的组合了。
继承需要添加类库：commons-logging-1.1.3.jar,  log4j-1.2.17.jar.（当然log4j自己的配置文件log4j.pproperties还是要有的）
之后我们就可以直接面向commons logging编程了。

commons Logging主要操作就是两个类：Log接口，LoggerFactory
使用：
private Log log = LogFactory.getLog(CLASS.class);
日志级别：fatal,error,warn,info,debug,trace
这就ok了。
另外可以参考：
commons.apache.org/proper/commons-logging/guide.html
http://singleant.iteye.com/blog/934593

#  SLF4J与Log4j整合使用 
SLF4j----Simple Logging Facade for Java:简单日志门户。是一种JCL体系。类似于apache commons Logging,但是比commons logging使用更方便，更为强大。它是由Log4j的作者开发的，还有用于取代log4j的logback.
使用SLF4J,通常由两种组合SLF4J+Log4j; SLF4J+logback,目前推荐的是SLF4J+logback.不过对于习惯使用log4j的我们也可以使用SLF4J+log4j.
同时SLF4J还支持android日志操作。
` SLF4J的体系：SLF4J-api,然后调用具体实现的包装库wrapper如：slf4j-log4j12-1.7.7.jar然后是调用具体的实现如Log4j-1.2.jar.所以通常我们需要导入这三个库。 `
使用SLF4J:的明显好处：
` 1.参数化日志书写形式：通过占位符{}来参数化： `
参数化对象
Object entry = new SomeObject();
logger.debug("The entry is {}.", entry);
参数化一个或多个字符串：
logger.debug("The new entry is {}.", entry);
logger.debug("The new entry is {}. It replaces {}.", entry, oldEntry);
关于输出{}相关问题转义等：
logger.debug("Set {1,2} differs from {}", "3");
输出"Set {1,2} differs from 3".
logger.debug("Set {1,2} differs from {{}}", "3");
输出"Set {1,2} differs from {3}".
logger.debug("Set \\{} differs from {}", "3");
输出 "Set {} differs from 3".
` 2.使用参数化方法。可以避免书写 isDebugEnabled,isInfoEnabled等方法。但是不使用参数化方法时还得看情况书写。 `
以下摘自slf4j-log4j12.jar中的源代码，看看就知道为什么会有这个有点和区分了：
摘自slf4j-log4j12.jar中的源代码
非参数化方法：
public void debug(String msg) {
    logger.log(FQCN, Level.DEBUG, msg, null);
  }
参数化方法：
  public void debug(String format, Object arg) {
    if (logger.isDebugEnabled()) {//参数化方法会自动判断log4j配置文件是否允许debug,如若不允许则直接忽略。
      FormattingTuple ft = MessageFormatter.format(format, arg);
      logger.log(FQCN, Level.DEBUG, ft.getMessage(), ft.getThrowable());
    }
  }
好了，不多说。
<wrap hi>需要导入的库：slf4j-api.jar,log4j-1.2.jar，slf4j-log4j12-1.7.7.jar通常还推荐加上slf4j-simple-1.7.7.jar这个默认的实现，以便在没有其他实现时不至于抛出异常。
使用SLF4j，它会自动寻找classpath路下的实现库，所以我们无须进行配置，但是log4j的配置文件log4j.properties还是要有的。
注意，为了避免冲突，classpath下只能有一种实现。否则将出现意想不到的的问题。</wrap>
maven集成:
<dependency>
 <groupId>org.slf4j</groupId>
 <artifactId>slf4j-api</artifactId>
 <version>1.7.7</version>
</dependency>
<dependency>
 <groupId>org.slf4j</groupId>
 <artifactId>slf4j-log4j12</artifactId>
 <version>1.7.7</version>
</dependency>
主要使用的API:Logger,LoggerFactory,
使用实例：
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
参考：http://www.slf4j.org/faq.html#logging_performance
http://www.importnew.com/7450.html
http://singleant.iteye.com/blog/934593