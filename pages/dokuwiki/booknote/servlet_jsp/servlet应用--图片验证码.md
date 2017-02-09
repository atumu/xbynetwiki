title: servlet应用--图片验证码 

#  servlet应用--图片验证码 
原文：http://my.oschina.net/june6502/blog/224287?p=1
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-182104.png)
```

package june;
/**验证码实现思路：
*在Servlet中随机产生验证码字符序列，并计入session中，
*JSP中以图片的形式进行显示。当用户在JSP表单中输入验证码并提交时，
*在相应的Servlet中验证是否与session中保存的验证码一致。
*/
import java.awt.*;
import java.awt.image.BufferedImage;
import java.util.Random;
import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.annotation.WebInitParam;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
//注释配置Servlet
@WebServlet(name = "StoreCode", urlPatterns = { "/storecode" }, initParams = {
  @WebInitParam(name = "width", value = "80"),
  @WebInitParam(name = "height", value = "30"),
  @WebInitParam(name = "codeCount", value = "4") })
public class StoreCode extends HttpServlet {
 private static final long serialVersionUID = 1L;
 private int width = 80; // 验证码图片的宽
 private int height = 30; // 验证码图片的高
 private int codeCount = 4; // 验证码图片的字符数
 private int x = 16;
 private int fontHeight=22;
 private int codeY=22;
 private final char[] codeSequence = { 'A', 'B', 'C', 'D', 'E', 'F', 'G',
   'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T',
   'U', 'V', 'W', 'X', 'Y', 'Z', '0', '1', '2', '3', '4', '5', '6',
   '7', '8', '9' };
  
 protected void doGet(HttpServletRequest request,
   HttpServletResponse response) throws ServletException,
   java.io.IOException {
  //构造一个类型为预定义图像类型之一的BufferedImage，设置图像的宽，高，和类型（TYPE_INT_RGB）
  BufferedImage Img = new BufferedImage(width, height,
    BufferedImage.TYPE_INT_RGB);
   
  // 返回Graphics2D，Graphics2D类扩展自Graphics 类，以提供对几何形状、坐标转换、颜色管理和文本布局更为复杂的控制
  Graphics g = Img.getGraphics();
   
  Random random = new Random();
  // 将图像填充为白色
  g.setColor(Color.WHITE);
  //填充指定的矩形。x,y坐标均为0，宽为width,高为height
  g.fillRect(0, 0, width, height);
  // 创建字体，字体的大小应该根据图片的高度来定。
  Font font = new Font("Times new Roman", Font.PLAIN, fontHeight);
  g.setColor(Color.black);
  g.setFont(font);
  Color juneFont = new Color(153, 204, 102);
  // 随机产生130条干扰线，不易被其它程序探测
  g.setColor( juneFont );
  for (int i = 0; i < 130; i++) {
    
   //返回伪随机数
   int x= random.nextInt(width);
   int y = random.nextInt(height);
   int xl = random.nextInt(16);  //80/5=16
   int yl = random.nextInt(16);
   //在此图形上下文的坐标系中，使用当前颜色在点 (x1, y1) 和 (x2, y2) 之间画一条线
   g.drawLine(x, y, x + xl, y + yl);
    
  }
  // randomCode用于保存随机产生的验证码，以便用户登录后进行验证,线程安全的可变字符序列
  StringBuffer randomCode = new StringBuffer();
  // 随机产生codeCount数字的验证码
  for (int i = 0; i < codeCount; i++) {
   //返回 char 参数的字符串表示形式
   String strRand = String.valueOf(codeSequence[random.nextInt(36)]);
    
   // 用随机产生的颜色将验证码绘制到图像中
   //创建具有指定红色、绿色和蓝色值的不透明的 sRGB 颜色，这些值都在 (0 - 255) 的范围内
   g.setColor(new Color(20 + random.nextInt(110), 20 + random
     .nextInt(110), 20 + random.nextInt(110)));
    
   //使用此图形上下文的当前字体和颜色绘制由指定 string 给定的文本。最左侧字符的基线位于此图形上下文坐标系的 (x, y) 位置处。
   g.drawString(strRand, (i + 1) * x-4, codeY);
   randomCode.append(strRand);
  }
  HttpSession session = request.getSession(); // 将四位数字的验证码保存到Session中
  session.setAttribute("realcode", randomCode.toString());
  // 禁止浏览器缓存
  response.setHeader("Pragma", "no-cache");    //HTTP   1.0
  response.setHeader("Cache-Control", "no-cache");//HTTP   1.1
  response.setDateHeader("Expires", 0);      //在代理服务器端防止缓冲
  response.setContentType("image/gif");      //设置正被发往客户端的响应的内容类型
   
  // 将图像输出到Servlet输出流中,ServletOutputStream提供了向客户端发送二进制数据的输出流
  ServletOutputStream sos = response.getOutputStream();
  ImageIO.write(Img, "gif", sos);           //使用支持给定格式的任意 ImageWriter 将一个图像写入 OutputStream
  sos.flush();                    //刷新此输出流并强制写出所有缓冲的输出字节
  sos.close();
 }
}

```
```

<%@ page language="java" contentType="text/html; charset=UTF-8"
 pageEncoding="UTF-8" import="java.util.*"%>
<%@ page import="java.io.PrintWriter" import="java.lang.String"%>
<!DOCTYPE HTML>
<html>
<head>
<title>验证码</title>
<meta charset="UTF-8">
<link rel="stylesheet" href="css/all.css">
</head>
<body>
 <div class="page-back">
  <div class="page-welcome">
   <span class="font-welcome">JSP作业5：Servlet应用</span>
  </div>
  <div class="page-main">
   <div class="page-check">
    <form name="form1" method="get" onsubmit="#">
     <ul>
      <li class="input-info">输入验证码：</li>
      <li><input type="text" id="input-code" name="input-code">
       <!-- 点击一下重新加载一次验证码 --> <input type="image" name="img-code"
       id="img-code" alt="看不清，点击换图" src="checkcode"
       onclick="javascript:this.src='checkcode?+rand=Math.random()'"></li>
      <li><input type="submit" value="验证" id="check-button"
       onclick="#"></li>
      <li><%
       String realCode = request.getSession().getAttribute("realcode").toString();
       if (request.getParameter("input-code") != null) {
        String userCode = request.getParameter("input-code");
        PrintWriter output = response.getWriter();//创建输出流对象
        if (userCode.equalsIgnoreCase(realCode)) {
         out.print("&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入正确");
        } else {
         out.print("&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入错误，请重新输入！");
        }
       }
      %>
      </li>
     </ul>
    </form>
   </div>
  </div>
  <div class="page-footer">
   <p>Copyright ? 2014 All Rights Reserved</p>
   <p>
    Author:June(王映君) &nbsp;&nbsp; <a
     href="http://my.oschina.net/june6502/blog">博客链接：</a>
   </p>
  </div>
 </div>
</body>
</html>

```
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-182238.png)