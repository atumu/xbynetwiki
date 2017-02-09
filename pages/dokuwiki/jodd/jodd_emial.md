title: jodd_emial 

#  Jodd发送Email 

开启后项目导入jodd的mail包就可以开始发送邮件了
```

import jodd.mail.Email;
import jodd.mail.SendMailSession;
import jodd.mail.SmtpServer;
import jodd.mail.SmtpSslServer;

public class Test {

	public static void main(String[] args) {
//		sendQQMail();
		send126Mail();
	}


	public static void sendQQMail(){
		Email email = Email.create()
				.from("123@qq.com")
				.to("123@126.com")
				.subject("testQQ")
				.addText("ab你好!cd")
				.addHtml("<html><META http-equiv=Content-Type content=\"text/html; charset=utf-8\">" +
				 "<body><h1>你好v</h1></body></html>");

				SendMailSession mailSession = 
						new SmtpSslServer("smtp.qq.com","1234566", "1212121")
							.createSession();
				mailSession.open();
				mailSession.sendMail(email);
				mailSession.close();
				System.out.println("发送QQ成功!...");
	}
	
	public static void send126Mail(){
		Email email = Email.create()
				.from("123@126.com")
				.to("23123@126.com")
				.subject("test126")
				.addHtml("<html><META http-equiv=Content-Type content=\"text/html; charset=utf-8\">" +
				 "<body>123123123")
				.addText("ab你好!cd")
				.addHtml("<h1>你好v</h1></body></html>");

				SendMailSession mailSession = 
						new SmtpServer("smtp.126.com","123123", "123123")
							.createSession();
				mailSession.open();
				mailSession.sendMail(email);
				mailSession.close();
				System.out.println("发送126成功!...");
	}
}

```