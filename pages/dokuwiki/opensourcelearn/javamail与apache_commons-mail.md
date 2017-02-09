title: javamail与apache_commons-mail 

#  javamail与apache_commons-mail 

<WRAP center round box 60%>
使用javaMail发送文本邮件和带附件邮件以及android后台发送邮件
</WRAP>

#  一、使用javamail发送邮件 

导入jar包
mail.jar activation.jar(JDK6.0以后官方API自带)
* **JDK6.0 以后开发 只需要导入 mail.jar**  ----- rt.jar 提供javax.activation 开发包
* JDK5.0（包括） 之前开发，需要导入mail.jar 和 activation.jar 
##  发送电子邮件 


主要步骤如下：

1,获取系统Properties.
Properties props = System.getProperties();
2,将您的SMTP服务器名添加到mail.smtp.host关键字的属性中.
Props.pout( “ mail.smtp.host ” ,host);
3,获取基于Properties Session对象.
Session session = Session.getDefaultInstance(props,null);
4,从Session创建一个MimeMessage.
MimeMessage message = new MimeMessage(session);
5,设置消息from域.
Message.setForm(new InternetAddress(from));
6,设置to域.
Message.addRecipient(Message.RecipientType.TO,new InternetAddress(to));
7,设置消息主题.
message.setSubject( “ HelloJavaMail ” );
8,设置消息内容.
Message.setText( “ Welcome to JavaMail ” );
9发送消息.
Transport.send(message);

要发送电子邮件，应用程序应准备一个 MimeMessage 对象，然后在 Transport 类上使用静态方法 send() 来发送该对象。邮件通过 JavaMail 会话对象创建。

import java.util.Properties;

import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.AddressException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

// ...
        Properties props = new Properties();
        Session session = Session.getDefaultInstance(props, null);

        String msgBody = "...";

        try {
            Message msg = new MimeMessage(session);
            msg.setFrom(new InternetAddress("admin@example.com", "Example.com Admin"));
            msg.addRecipient(Message.RecipientType.TO,
                             new InternetAddress("user@example.com", "Mr. User"));
            msg.setSubject("Your Example.com account has been activated");
            msg.setText(msgBody);
            Transport.send(msg);

        } catch (AddressException e) {
            // ...
        } catch (MessagingException e) {
            // ...
        }


发件人和收件人

发件人和收件人电子邮件地址在 JavaMail 中使用 InternetAddress 类的实例表示。构造函数采用字符串形式的电子邮件地址，如果该地址并非有效的电子邮件地址，则会引发 AddressException。或者，您可以提供字符串形式的个人姓名作为第二个参数。

要设置发件人地址，该应用程序应调用 MimeMessage 对象的 setFrom() 方法。

收件人类型可以是Message.RecipientType.TO、Message.RecipientType.CC 或 Message.RecipientType.BCC 中的任何一种。

您可以使用 setReplyTo() 方法设置“回复”地址。

邮件和标头

您可以通过调用 MimeMessage 对象的方法建立邮件的内容。setSubject() 方法用于设置主题，而 setText() 方法用于设置（纯文本）正文内容。

出于安全方面的考虑，邮件服务不允许在传出电子邮件上使用任意标头。 邮件服务将覆盖某些标头，如邮件发送日期。添加到传出邮件的其他标头将被删除。


##  二、发送带附件的邮件： 

您发送的邮件中除了可以包含纯文本邮件正文以外，还可以包含文件附件或 HTML 邮件正文。您需要创建一个要包含各部分的 MimeMultipart 对象，然后为每个附件或备用邮件正文创建一个 MimeBodyPart 对象，并将其添加到容器中。最后，将该容器分配到 MimeMessage 的内容。

import javax.activation.DataHandler;
import javax.mail.Multipart;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMultipart;

// ...
        String htmlBody;        // ...
        byte[] attachmentData;  // ...

        Multipart mp = new MimeMultipart();

        MimeBodyPart htmlPart = new MimeBodyPart();
        htmlPart.setContent(htmlBody, "text/html");
        mp.addBodyPart(htmlPart);

        MimeBodyPart attachment = new MimeBodyPart();
        attachment.setFileName("manual.pdf");
        attachment.setContent(attachmentData, "application/pdf");
        mp.addBodyPart(attachment);

        message.setContent(mp);

##  三。接收邮件： 

Store和folder用session获取消息,与发送消息开始很相似,但是在session得到后,很可能实用用户名和密码或实用Authenticator连接到一个Store.类似于Transport,也是一样要告诉store用什么协议.例如
Store store = session.getStore( “ pop3 ” );
Store.connect(host,username,password);
连接到Store之后,接下来,获得一个folder,必须打开它就可以读取里边的消息了.
Folder folder = store.getFolder("INBOX");
folder.open(Folder.READ_ONLY);
Message[] message = folder.getMessages();
POP3唯一可用的文件夹就是INBOX,如果实用IMAP,还可以用其他的文件夹.
当读到了具体的message以后,就可以用getContent来获取内容,或者用writeTo()将内容写入流,getContent()方法只能得到消息内容,而writeTo()的输出却包含消息头.
System.out.println(((MimeMessage)message).getConntent());
一旦读取完毕邮件,要关闭store和folder的连接.
folder.colse(boolean);
store.colse();
传递给folder的close()方法的boolean参数表示是否清楚已删除的消息从而更新folder.

##  四、简单示例 

{
    // 发送邮件的服务器的IP和端口    
    private String mailServerHost;    
    private String mailServerPort = "";   

    // 邮件发送者的地址    
    private String fromAddress;    
    // 邮件接收者的地址    
    private String toAddress;    
    // 登陆邮件发送服务器的用户名和密码    
    private String userName;    
    private String password;    
    // 是否需要身份验证    
    private boolean validate = true;    
    // 邮件主题    
    private String subject;    
    // 邮件的文本内容    
    private String content;    
    // 邮件附件的文件名    
    private String[] attachFileNames;      
} 
public boolean sendTextMail(MailSenderInfo mailInfo) 
    {
        // 判断是否需要身份认证    
        MyAuthenticator authenticator = null;    
        Properties pro = mailInfogetProperties();   
        if (mailInfoisValidate()) 
        {    
            // 如果需要身份认证则创建一个密码验证器    
            authenticator = new MyAuthenticator(mailInfogetUserName() mailInfogetPassword());    
        }   
        // 根据邮件会话属性和密码验证器构造一个发送邮件的session    
        Session sendMailSession = SessiongetDefaultInstance(proauthenticator);    
        try 
        {    
            // 根据session创建一个邮件消息    
            Message mailMessage = new MimeMessage(sendMailSession);    
            // 创建邮件发送者地址    
            Address from = new InternetAddress(mailInfogetFromAddress());    
            // 设置邮件消息的发送者    
            mailMessagesetFrom(from);    
            // 创建邮件的接收者地址并设置到邮件消息中    
            Address to = new InternetAddress(mailInfogetToAddress());    
            mailMessagesetRecipient(MessageRecipientTypeTOto);    
            // 设置邮件消息的主题    
            mailMessagesetSubject(mailInfogetSubject());    
            // 设置邮件消息发送的时间    
            mailMessagesetSentDate(new Date());    
            // 设置邮件消息的主要内容    
            String mailContent = mailInfogetContent();    
            mailMessagesetText(mailContent);    
            // 发送邮件    
            Transportsend(mailMessage);   
            return true;    
        } 
        catch (MessagingException ex) 
        {    
            exprintStackTrace();    
        }    
        return false;    
    }


##  五、android开发 

只需要导入javamail包即可:
activation.jar
additionnal.jar
mail.jar
##  六、JavaMail综合示例 

    Session类定义了一个基本的邮件会话,所有的其他类都是由这个session才得意生效的,Session对象用java.util.Properties对象获取信息,如邮件服务器,用户名,密码及整个应用程序中共享的其他信息.类的构造器是此有的,private.它能用getDefaultInstance()方法来共享.获取Session对象的方方法如下:
Properties props = new Properties();
Session session = Session.getDefaultInstance(props,null);
Null参数都是Authenticator对象,在这里没有使用.
对于大多数情况,共享的session已经足够用了.

    Message消息类,在获得了Session对象后,就可以继续创建要发送的消息.因为Message是个抽象类,您必须用一个子类,多数情况下为java.mail.internet.MimeMessage.这个能理解成MIME类型和头的电子邮件消息.正如不同的RFC中定义的,虽然在某些头部域非ASCII字符也能被编译,但是Message头只能被限制用US-ASCII字符.要创建一个Message请将Session对象传递给MimeMessage的构造器.
MimeMessage message = newMimeMessage(session);
一旦获得消息,就可以设置各个部分了.最基本的就是setContent()方法,例如/
message.setContent( “ Hello ” , ” text/plain ” );
如果知道在实用MimeMessage,而且消息是纯文本格式,就可以用setText()方法,它只需要代表实际内容的参数.(Mime类型缺省为text/plain)
用setSubject()方法设置subject(主题);
message.setSubject( “ 主题 ” );

    Address地址类,和Message一样也是一个抽象类,一旦创建了Session和Message并将内容填入消息后,就可以用Address确定信件的地址了,用javax.mail.internet.
InternetAddress类.若创建的地址只包含电子邮件地址,只要传递电子邮件地址给构造器就可以了.例如:Address address = new InternetAddress( “ it5719@163.com ” );
若希望名字挨着电子邮件现实,就可以把它传递给构造器,如下:
Address address = new InternetAddress( “ it5719@163.com ” , ” 我心依旧 ” );

需要为消息的from域和to域创建地址对象,除非邮件服务器阻止,没有什么能阻止你发送一段看上去是任何人的消息了呵呵.一旦创建address将他们域消息连接方法有两种,如要要识别发件人的就可以用setFrom()和setReplyTo方法.然后message.setFrom(address);
需要实用多个from地址的就用addFrom()方法.例子如下:
Address[] address = ,.,. ;    message.addFrom(address);
若要识别消息recipient收件人,就要实用addRecipient()方法了.例如:
message.addRecipient(type,address)

    Authenticator与java.net类一样,JavaMailAPI也可以利用Authentcator通过用户名密码访问受保护的资源.对于JavaMail来说,这些资源就是邮件服务器,Authentcator类在javax.mail包中.要使用Authenticator,首先创建一个抽象的子类,并从
getPasswordAuthentication方法中返回passwordAuthentication实例,创建完成后,您必须向session注册Authenticator,然后在需要认证的时候会通知它,其实说白了就是把配置的用户名和密码返回给调用它的程序.例如:
Properties props = new properties();
Authenticator auth = new MailAuthenticator()//接口声明,创建自己新类的实例.
Session session = Session.getDefauItInstance(props,auth);

    Transport消息发送传输类,这个类用协议指定的语言发送消息,通常是SMTP,它是抽象类,它的工作方式与Session有些类似,尽调用静态方法send()方法,就OK了.例如:
Transport.send(message);

或者也可以从针对协议的会话中获取一个特定的实例,传递用户名和密码.发送消息,然后关闭连接,例如:
message.saveChanges();
transport transport = session.getTreansport( “ smtp ” );//指定的协议
transport.connect(host,username,password);
transport.sendMessage(message,message.getAllRecipients());
transport.close();


如果要观察传到邮件服务器上的邮件命令,请用session.setDubug(true)设置调试标志.

    Store和folder用session获取消息,与发送消息开始很相似,但是在session得到后,很可能实用用户名和密码或实用Authenticator连接到一个Store.类似于Transport,也是一样要告诉store用什么协议.例如
Store store = session.getStore( “ pop3 ” );
Store.connect(host,username,password);
连接到Store之后,接下来,获得一个folder,必须打开它就可以读取里边的消息了.
Folder folder = store.getFolder("INBOX");
folder.open(Folder.READ_ONLY);
Message[] message = folder.getMessages();
POP3唯一可用的文件夹就是INBOX,如果实用IMAP,还可以用其他的文件夹.
当读到了具体的message以后,就可以用getContent来获取内容,或者用writeTo()将内容写入流,getContent()方法只能得到消息内容,而writeTo()的输出却包含消息头.
System.out.println(((MimeMessage)message).getConntent());
一旦读取完毕邮件,要关闭store和folder的连接.
folder.colse(boolean);
store.colse();
传递给folder的close()方法的boolean参数表示是否清楚已删除的消息从而更新folder.
    上面就是JavaMail邮件操作的基本的常用类,我觉得理解了这几个类的机制,基本就可以处理一般的邮件操作了.
首先是一个Authenticator类的实现:记录用户名和密码:
import javax.mail.*;
 
 
public class MailAuthenticator extends Authenticator
{
    //******************************
    //由于发送邮件的地方比较多,
    //下面统一定义用户名,口令.
    //******************************
    public static String HUAWEI_MAIL_USER = "it5719@163.com";
    public static String HUAWEI_MAIL_PASSWORD = "密码";
 
 
    public MailAuthenticator()
    {
    }
 
    protected PasswordAuthentication getPasswordAuthentication()
    {
        return new PasswordAuthentication(HUAWEI_MAIL_USER, HUAWEI_MAIL_PASSWORD);
    }
 
}
 
这个类是发送邮件的类.
 
 
package com.deepdo.common.mail;
 
/**
 * 此处插入类型说明。
 * 创建日期：(2006-4-21 14:57:16)
 * @author：张宏亮
 */
 
import java.util.*;
import java.io.*;
import javax.mail.*;
import javax.mail.internet.*;
import javax.activation.*;
 
 
public class SendMail {
 
    //要发送Mail地址
    private String mailTo = null;
    //Mail发送的起始地址
    private String mailFrom = null;
    //SMTP主机地址
    private String smtpHost = null;
    //是否采用调试方式
    private boolean debug = false;
 
    private String messageBasePath = null;
    //Mail主题
    private String subject;
    //Mail内容
    private String msgContent;
 
    private Vector attachedFileList;
    private String mailAccount = null;
    private String mailPass = null;
    private String messageContentMimeType ="text/html; charset=gb2312";
 
    private String mailbccTo = null;
    private String mailccTo = null;
    /**
     * SendMailService 构造子注解。
     */
    public SendMail() {
        super();
   
    }
 
    private void fillMail(Session session,MimeMessage msg) throws IOException, MessagingException{
   
        String fileName = null;
        Multipart mPart = new MimeMultipart();
        if (mailFrom != null) {
            msg.setFrom(new InternetAddress(mailFrom));
            System.out.println("发送人Mail地址："+mailFrom);
        } else {
            System.out.println("没有指定发送人邮件地址！");
            return;
        }
        if (mailTo != null) {
            InternetAddress[] address = InternetAddress.parse(mailTo);
            msg.setRecipients(Message.RecipientType.TO, address);
            System.out.println("收件人Mail地址："+mailTo);
        } else {
            System.out.println("没有指定收件人邮件地址！");
            return;
        }
   
        if (mailccTo != null) {
            InternetAddress[] ccaddress = InternetAddress.parse(mailccTo);
            System.out.println("CCMail地址："+mailccTo);
            msg.setRecipients(Message.RecipientType.CC, ccaddress);
        }
        if (mailbccTo != null) {
            InternetAddress[] bccaddress = InternetAddress.parse(mailbccTo);
            System.out.println("BCCMail地址："+mailbccTo);
            msg.setRecipients(Message.RecipientType.BCC, bccaddress);
        }
        msg.setSubject(subject);
        InternetAddress[] replyAddress = { new InternetAddress(mailFrom)};
        msg.setReplyTo(replyAddress);
        // create and fill the first message part
        MimeBodyPart mBodyContent = new MimeBodyPart();
        if (msgContent != null)
            mBodyContent.setContent(msgContent, messageContentMimeType);
        else
            mBodyContent.setContent("", messageContentMimeType);
        mPart.addBodyPart(mBodyContent);
        // attach the file to the message
        if (attachedFileList != null) {
            for (Enumeration fileList = attachedFileList.elements(); fileList.hasMoreElements();) {
                fileName = (String) fileList.nextElement();
                MimeBodyPart mBodyPart = new MimeBodyPart();
   
                // attach the file to the message
                FileDataSource fds = new FileDataSource(messageBasePath + fileName);
                System.out.println("Mail发送的附件为："+messageBasePath + fileName);
                mBodyPart.setDataHandler(new DataHandler(fds));
                mBodyPart.setFileName(fileName);
                mPart.addBodyPart(mBodyPart);
            }
        }
        msg.setContent(mPart);
        msg.setSentDate(new Date());
    }
    /**
     * 此处插入方法说明。
     */
    public void init()
    {
   
    }
    /**
     * 发送e_mail，返回类型为int
     * 当返回值为0时，说明邮件发送成功
     * 当返回值为3时，说明邮件发送失败
     */
    public int sendMail() throws IOException, MessagingException {
   
        int loopCount;
        Properties props = System.getProperties();
        props.put("mail.smtp.host", smtpHost);
        props.put("mail.smtp.auth", "true");
   
        MailAuthenticator auth = new MailAuthenticator();
   
        Session session = Session.getInstance(props, auth);
        session.setDebug(debug);
        MimeMessage msg = new MimeMessage(session);
        Transport trans = null;
        try {
   
            fillMail(session,msg);
            // send the message
            trans = session.getTransport("smtp");
            try {
                trans.connect(smtpHost, MailAuthenticator.HUAWEI_MAIL_USER, MailAuthenticator.HUAWEI_MAIL_PASSWORD);//, HUAWEI_MAIL_PASSWORD);
            } catch (AuthenticationFailedException e) {
                e.printStackTrace();
                System.out.println("连接邮件服务器错误：");
                return 3;
            } catch (MessagingException e) {
                System.out.println("连接邮件服务器错误：");
                return 3;
            }
   
            trans.send(msg);
            trans.close();
   
        } catch (MessagingException mex) {
            System.out.println("发送邮件失败：");
            mex.printStackTrace();
            Exception ex = null;
            if ((ex = mex.getNextException()) != null) {
                System.out.println(ex.toString());
                ex.printStackTrace();
            }
            return 3;
        } finally {
            try {
                if (trans != null && trans.isConnected())
                    trans.close();
            } catch (Exception e) {
                System.out.println(e.toString());
            }
        }
        System.out.println("发送邮件成功！");
        return 0;
    }
    public void setAttachedFileList(java.util.Vector filelist)
    {
        attachedFileList = filelist;
    }
    public void setDebug(boolean debugFlag)
    {
        debug=debugFlag;
    }
    public void setMailAccount(String strAccount) {
        mailAccount = strAccount;
    }
    public void setMailbccTo(String bccto) {
        mailbccTo = bccto;
    }
    public void setMailccTo(String ccto) {
        mailccTo = ccto;
    }
    public void setMailFrom(String from)
    {
        mailFrom=from;
    }
    public void setMailPass(String strMailPass) {
        mailPass = strMailPass;
    }
    public void setMailTo(String to)
    {
        mailTo=to;
    }
    public void setMessageBasePath(String basePath)
    {
        messageBasePath=basePath;
    }
    public void setMessageContentMimeType(String mimeType)
    {
        messageContentMimeType = mimeType;
    }
    public void setMsgContent(String content)
    {
        msgContent=content;
    }
    public void setSMTPHost(String host)
    {
        smtpHost=host;
    }
    public void setSubject(String sub)
    {
        subject=sub;
    }
   
    public static void main(String[] argv) throws Exception
    {
        for(int i = 0;i<10;i++) {
        SendMail sm = new SendMail();
        sm.setSMTPHost("SMTP地址");
        sm.setMailFrom("发送地址");
        sm.setMailTo("目标地址");
        sm.setMsgContent("内容");
        sm.setSubject("标题");
        sm.sendMail();
        }
    }
}


##  高级示例： 

 
import java.io.ByteArrayInputStream;  
import java.io.IOException;  
import java.io.InputStream;  
import java.io.OutputStream;  
import java.security.Security;  
import java.util.Properties;  
  
import javax.activation.DataHandler;  
import javax.activation.DataSource;  
import javax.mail.Authenticator;  
import javax.mail.Message;  
import javax.mail.PasswordAuthentication;  
import javax.mail.Session;  
import javax.mail.Transport;  
import javax.mail.internet.InternetAddress;  
import javax.mail.internet.MimeMessage;  
  
public class GMailSender extends Authenticator {  
    private String mailhost = "smtp.gmail.com";  
    private String user;  
    private String password;  
    private Session session;  
  
    static {  
        Security.addProvider(new JSSEProvider());  
    }  
  
    public GMailSender(String user, String password) {  
        this.user = user;  
        this.password = password;  
  
        Properties props = new Properties();  
        props.setProperty("mail.transport.protocol", "smtp");  
        props.setProperty("mail.host", mailhost);  
        props.put("mail.smtp.auth", "true");  
        props.put("mail.smtp.port", "465");  
        props.put("mail.smtp.socketFactory.port", "465");  
        props.put("mail.smtp.socketFactory.class",  
                "javax.net.ssl.SSLSocketFactory");  
        props.put("mail.smtp.socketFactory.fallback", "false");  
        props.setProperty("mail.smtp.quitwait", "false");  
  
        session = Session.getDefaultInstance(props, this);  
    }  
  
    protected PasswordAuthentication getPasswordAuthentication() {  
        return new PasswordAuthentication(user, password);  
    }  
  
    public synchronized void sendMail(String subject, String body,  
            String sender, String recipients) throws Exception {  
        try {  
            MimeMessage message = new MimeMessage(session);  
            DataHandler handler = new DataHandler(new ByteArrayDataSource(  
                    body.getBytes(), "text/plain"));  
            message.setSender(new InternetAddress(sender));  
            message.setSubject(subject);  
            message.setDataHandler(handler);  
            if (recipients.indexOf(',') > 0)  
                message.setRecipients(Message.RecipientType.TO,  
                        InternetAddress.parse(recipients));  
            else  
                message.setRecipient(Message.RecipientType.TO,  
                        new InternetAddress(recipients));  
            Transport.send(message);  
        } catch (Exception e) {  
  
        }  
    }  
  
    public class ByteArrayDataSource implements DataSource {  
        private byte[] data;  
        private String type;  
  
        public ByteArrayDataSource(byte[] data, String type) {  
            super();  
            this.data = data;  
            this.type = type;  
        }  
  
        public ByteArrayDataSource(byte[] data) {  
            super();  
            this.data = data;  
        }  
  
        public void setType(String type) {  
            this.type = type;  
        }  
  
        public String getContentType() {  
            if (type == null)  
                return "application/octet-stream";  
            else  
                return type;  
        }  
  
        public InputStream getInputStream() throws IOException {  
            return new ByteArrayInputStream(data);  
        }  
  
        public String getName() {  
            return "ByteArrayDataSource";  
        }  
  
        public OutputStream getOutputStream() throws IOException {  
            throw new IOException("Not Supported");  
        }  
    }  
  
}  


[java] view plaincopy
import java.security.AccessController;  
import java.security.Provider;  
  
  
public class JSSEProvider extends Provider {  
public JSSEProvider() {  
        super("HarmonyJSSE", 1.0, "Harmony JSSE Provider");  
        AccessController.doPrivileged(new java.security.PrivilegedAction<Void>() {  
            public Void run() {  
                put("SSLContext.TLS",  
                        "org.apache.harmony.xnet.provider.jsse.SSLContextImpl");  
                put("Alg.Alias.SSLContext.TLSv1", "TLS");  
                put("KeyManagerFactory.X509",  
                        "org.apache.harmony.xnet.provider.jsse.KeyManagerFactoryImpl");  
                put("TrustManagerFactory.X509",  
                        "org.apache.harmony.xnet.provider.jsse.TrustManagerFactoryImpl");  
                return null;  
            }  
        });  
    }  
}  



程序中调用：

GMailSender sender = new GMailSender("....@gmail.com", "password");  
            try {  
                sender.sendMail("This is Subject",     
                        "This is Body",     
                        "sender",     
                        ".....@qq.com");  
            } catch (Exception e) {  
                // TODO Auto-generated catch block  
                Log.e("other", e.getMessage(), e);     
            }     

附件：
javamail1.5.jar工具包：http://pan.baidu.com/s/1Fu7UW
javamail-simple-sample：http://pan.baidu.com/s/1Ga6MP

本文链接：http://blog.csdn.net/xby1993/article/details/12713949

参考文章：
https://developers.google.com/appengine/docs/java/mail/usingjavamail?hl=zh-cn
http://www.blogjava.net/action/archive/2006/04/24/42794.html
http://www.open-open.com/lib/view/open1374292814222.html
http://blog.csdn.net/ming_light/article/details/9293215

#  apache commons学习系列之Email组件 
基于commons Email version 1.3
##  主要类： 

  SimpleEmail - This class is used to send basic text based emails.
   MultiPartEmail - This class is used to send multipart messages. This allows a text message with attachments either inline or attached.
   HtmlEmail - This class is used to send HTML formatted emails. It has all of the capabilities as MultiPartEmail allowing attachments to be easily added. It also supports embedded images.
   ImageHtmlEmail - This class is used to send HTML formatted emails with inline images. It has all of the capabilities as HtmlEmail but transform all image references to inline images.
   EmailAttachment - This is a simple container class to allow for easy handling of attachments. It is for use with instances of MultiPartEmail and HtmlEmail.
还有一个jdk中的类DefaultAuthenticator，用与用户验证
##  应用示例： 

###  SimpleEmail发送普通文本邮件： 

Email email = new SimpleEmail();
email.setHostName("smtp.googlemail.com");
email.setSmtpPort(465);
email.setAuthenticator(new DefaultAuthenticator("username", "password"));
email.setSSLOnConnect(true);//使用SSL加密发送
email.setFrom("user@gmail.com");
email.setSubject("TestMail");
email.setMsg("This is a test mail ... :-)");
email.addTo("foo@bar.com");//使用addTo可以添加多个目的地
email.send();
###  使用MultiPartEmail发送带附件的邮件： 

  // Create the attachment
  EmailAttachment attachment = new EmailAttachment();
//路径可以为绝对或相对地址，也可以使用setURL
  attachment.setPath("mypictures/john.jpg");
//附件处理方式，作为附件或内嵌为内容
  attachment.setDisposition(EmailAttachment.ATTACHMENT);
// attachment.setURL(new URL("http://www.apache.org/images/asf_logo_wide.gif"));

 attachment.setDescription("Picture of John");
  attachment.setName("John");

  // Create the email message
  MultiPartEmail email = new MultiPartEmail();
  email.setHostName("mail.myserver.com");
  email.addTo("jdoe@somewhere.org", "John Doe");
  email.setFrom("me@apache.org", "Me");
  email.setSubject("The picture");
  email.setMsg("Here is the picture you wanted");

  // add the attachment
  email.attach(attachment);

  // send the email
  email.send();
###  发送HtmlEmail: 

// Create the email message
 HtmlEmail email = new HtmlEmail();
 email.setHostName("mail.myserver.com");
 email.addTo("jdoe@somewhere.org", "John Doe");
 email.setFrom("me@apache.org", "Me");
 email.setSubject("Test email with inline image");
 // embed the image and get the content id
 URL url = new URL("http://www.apache.org/images/asf_logo_wide.gif");
 String cid = email.embed(url, "Apache logo");//使用内嵌图片
 // set the html message
 email.setHtmlMsg("<html>The apache logo - <img src=\"cid:"+cid+"\"></html>");
 // set the alternative message
 email.setTextMsg("Your email client does not support HTML messages");
 // send the email
 email.send();
也可以使用如下相对路径形式使用ImageHtml：

 // load your HTML email template
  String htmlEmailTemplate = ....

  // define you base URL to resolve relative resource locations
  URL url = new URL("http://www.apache.org");
  // create the email message
  HtmlEmail email = new ImageHtmlEmail();
//指定html模板中的baseUrl
 email.setDataSourceResolver(new DataSourceUrlResolverl(url));
  email.setHostName("mail.myserver.com");
  email.addTo("jdoe@somewhere.org", "John Doe");
  email.setFrom("me@apache.org", "Me");
  email.setSubject("Test email with inline image");
  // set the html message
  email.setHtmlMsg(htmlEmailTemplate);
  // set the alternative message
  email.setTextMsg("Your email client does not support HTML messages");
  // send the email
  email.send();
##  补充几点： 

1.开启email组件的调试，以便出现问题时能够sysout相关信息。
email.setDebug(true);
2.邮件安全：
开启STARTTLS or SSL检察:
Email.setSSLCheckServerIdentity(true)
开启：STARTTLS:
Email.setStartTLSRequired(true)
开启SSL连接支持
email.setSSLOnConnect(true);

