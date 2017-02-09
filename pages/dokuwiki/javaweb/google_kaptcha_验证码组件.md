title: google_kaptcha_验证码组件 

#  google kaptcha 验证码组件 
**kaptcha 是一个非常实用的验证码生成工具**。有了它，你可以生成各种样式的验证码，**因为它是可配置的**。kaptcha工作的原理是调用 ` com.google.code.kaptcha.servlet.KaptchaServlet `，生成一个图片。同时将生成的**验证码字符串放到 HttpSession中。**

使用kaptcha可以方便的配置：
  * 验证码的字体
  * 验证码字体的大小
  * 验证码字体的字体颜色
  * 验证码内容的范围(数字，字母，中文汉字！)
  * 验证码图片的大小，边框，边框粗细，边框颜色
  * 验证码的干扰线(可以自己继承com.google.code.kaptcha.NoiseProducer写一个自定义的干扰线)
  * 验证码的样式(鱼眼样式、3D、普通模糊……当然也可以继承com.google.code.kaptcha.GimpyEngine自定义样式)

官网：http://code.google.com/p/kaptcha/
```

<!-- kaptcha验证码 -->
		<dependency>
		  <groupId>com.google.code.kaptcha</groupId>
		  <artifactId>kaptcha</artifactId>
		  <version>2.3.2</version>
		</dependency>

```
##  使用方式一：通过配置web.xml使用 
配置web.xml文件：
```

<!--Kaptcha 验证码  -->
    <servlet>  
        <servlet-name>kaptcha</servlet-name>  
        <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>  
        <init-param>  
            <param-name>kaptcha.border</param-name>  
            <param-value>no</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.border.color</param-name>  
            <param-value>105,179,90</param-value>  
        </init-param>       
        <init-param>  
            <param-name>kaptcha.textproducer.font.color</param-name>  
            <param-value>red</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.image.width</param-name>  
            <param-value>250</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.image.height</param-name>  
            <param-value>90</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.textproducer.font.size</param-name>  
            <param-value>70</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.session.key</param-name>  
            <param-value>code</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.textproducer.char.length</param-name>  
            <param-value>4</param-value>  
        </init-param>  
        <init-param>  
            <param-name>kaptcha.textproducer.font.names</param-name>  
            <param-value>宋体,楷体,微软雅黑</param-value>  
        </init-param>       
    </servlet> 

  <servlet-mapping>  
<servlet-name>kaptcha</servlet-name>  
<url-pattern>/ClinicCountManager/kaptcha.jpg</url-pattern>  
lt;/servlet-mapping>  

```
```

<table>  
        <tr>  
            <td><img src="/ClinicCountManager/kaptcha.jpg"></td>  
            <td valign="top">  
          
                <form method="POST">  
                    <br>sec code:<input type="text" name="kaptchafield"><br />  
                    <input type="submit" name="submit">  
                </form>  
            </td>  
        </tr>  
    </table>    
  
    <br /><br /><br /><br />  
      
    <%  
        String c = (String)session.getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);  
        String parm = (String) request.getParameter("kaptchafield");  
          
        out.println("Parameter: " + parm + " ? Session Key: " + c + " : ");  
          
        if (c != null && parm != null) {  
            if (c.equals(parm)) {  
                out.println("<b>true</b>");  
            } else {  
                out.println("<b>false</b>");  
            }  
          
    %>  

```

方式二：springmvc配置kaptcha:
 上面的配置在普通jsp环境下面是有效的，如果在spring mvc环境下，则取不到session值，对于sping mvc环境验证码配置如下：
１.不用在web.xml进行相关配置，**在applicationContext.xml中配置**
```

<bean id="captchaProducer" class="com.google.code.kaptcha.impl.DefaultKaptcha">  
        <property name="config">  
            <bean class="com.google.code.kaptcha.util.Config">  
                <constructor-arg>  
                    <props>  
                        <prop key="kaptcha.border">no</prop>  
                        <prop key="kaptcha.border.color">105,179,90</prop>  
                        <prop key="kaptcha.textproducer.font.color">red</prop>  
                        <prop key="kaptcha.image.width">250</prop>  
                        <prop key="kaptcha.textproducer.font.size">90</prop>  
                        <prop key="kaptcha.image.height">90</prop>  
                        <prop key="kaptcha.session.key">code</prop>  
                        <prop key="kaptcha.textproducer.char.length">4</prop>  
                        <prop key="kaptcha.textproducer.font.names">宋体,楷体,微软雅黑</prop>  
                    </props>  
                </constructor-arg>  
            </bean>  
        </property>  
    </bean>  

```
目前有个项目的配置如下：
```

<bean id="captchaProducer" class="com.google.code.kaptcha.impl.DefaultKaptcha">  
        <property name="config">  
            <bean class="com.google.code.kaptcha.util.Config">  
                <constructor-arg>  
                    <props>  
                        <prop key="kaptcha.border">yes</prop>  
                        <prop key="kaptcha.border.color">105,179,90</prop>  
                        <prop key="kaptcha.textproducer.font.color">blue</prop>  
                        <prop key="kaptcha.image.width">118</prop>  
                        <prop key="kaptcha.image.height">40</prop>  
                        <prop key="kaptcha.textproducer.font.size">30</prop>
                        <prop key="kaptcha.noise.color">blue</prop>   
                        <prop key="kaptcha.session.key">code</prop>  
                        <prop key="kaptcha.textproducer.char.length">4</prop>  
                        <prop key="kaptcha.textproducer.font.names">宋体,楷体,微软雅黑</prop>
                        <prop key="kaptcha.noise.impl">com.google.code.kaptcha.impl.NoNoise</prop>  
                    </props>  
                </constructor-arg>  
            </bean>  
        </property>  
    </bean>	

```
```

import java.awt.image.BufferedImage;  
import javax.imageio.ImageIO;  
import javax.servlet.ServletOutputStream;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.servlet.ModelAndView;  
import com.google.code.kaptcha.Constants;  
import com.google.code.kaptcha.Producer;  

@Controller  
@RequestMapping("/")  
public class CaptchaImageCreateController {  
      
    private Producer captchaProducer = null;  
  
    @Autowired  
    public void setCaptchaProducer(Producer captchaProducer) {  
        this.captchaProducer = captchaProducer;  
    }  
  
    @RequestMapping("/captcha-image")  
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {  
  
        response.setDateHeader("Expires", 0);  
        // Set standard HTTP/1.1 no-cache headers.  
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");  
        // Set IE extended HTTP/1.1 no-cache headers (use addHeader).  
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");  
        // Set standard HTTP/1.0 no-cache header.  
        response.setHeader("Pragma", "no-cache");  
        // return a jpeg  
        response.setContentType("image/jpeg");  
        // create the text for the image  
        String capText = captchaProducer.createText();  
        // store the text in the session  
        request.getSession().setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);  
        // create the image with the text  
        BufferedImage bi = captchaProducer.createImage(capText);  
        ServletOutputStream out = response.getOutputStream();  
        // write the data out  
        ImageIO.write(bi, "jpg", out);  
        try {  
            out.flush();  
        } finally {  
            out.close();  
        }  
        return null;  
    }  
  @RequestMapping("/captcha/valid")
    @ResponseBody
    public String validCaptcha(HttpServletRequest request, HttpServletResponse response,@RequestParam("captchaCode")String captchaCode){
    	HttpSession session = request.getSession();
    	String code = (String)session.getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);
		if(StringUtils.checkEquals(code, captchaCode)){
			return "true";
		}
    	return "false";
    }
  
}  

```
```

<div class="chknumber">  
       <label>验证码：          
       <input name="kaptcha" type="text" id="kaptcha" maxlength="4" class="chknumber_input" />               
       </label>  
        <img src="/ClinicCountManager/captcha-image.do" width="55" height="20" id="kaptchaImage"  style="margin-bottom: -3px"/>   
       <script type="text/javascript">      
        $(function(){           
            $('#kaptchaImage').click(function () {//生成验证码  
             $(this).hide().attr('src', '/ClinicCountManager/captcha-image.do?' + Math.floor(Math.random()*100) ).fadeIn(); })      
                  });   
        
       </script>   
     </div> 

```
如果想设置点击图片更换验证码，可以加上如下js，需要jquery
```

function changeCode() {  
    $('#kaptchaImage').hide().attr('src', './kaptcha/getKaptchaImage.do?' + Math.floor(Math.random()*100) ).fadeIn();  
    event.cancelBubble=true;  
}  

```
 取验证码的方式
```

String code = (String)session.getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);

```  
      
 如果需要全部数字
```

<init-param>       
    <param-name>kaptcha.textproducer.char.string</param-name>       
    <param-value>0123456789</param-value>       
</init-param>

```  
 去掉干扰线
```

<init-param>  
    <param-name>kaptcha.noise.impl</param-name>  
    <param-value>com.google.code.kaptcha.impl.NoNoise </param-value>  
</init-param>

```

##  kaptcha可配置项 
```

kaptcha.border  是否有边框  默认为true  我们可以自己设置yes，no  
kaptcha.border.color   边框颜色   默认为Color.BLACK  
kaptcha.border.thickness  边框粗细度  默认为1  
kaptcha.producer.impl   验证码生成器  默认为DefaultKaptcha  
kaptcha.textproducer.impl   验证码文本生成器  默认为DefaultTextCreator  
kaptcha.textproducer.char.string   验证码文本字符内容范围  默认为abcde2345678gfynmnpwx  
kaptcha.textproducer.char.length   验证码文本字符长度  默认为5  
kaptcha.textproducer.font.names    验证码文本字体样式  默认为new Font("Arial", 1, fontSize), new Font("Courier", 1, fontSize)  
kaptcha.textproducer.font.size   验证码文本字符大小  默认为40  
kaptcha.textproducer.font.color  验证码文本字符颜色  默认为Color.BLACK  
kaptcha.textproducer.char.space  验证码文本字符间距  默认为2  
kaptcha.noise.impl    验证码噪点生成对象  默认为DefaultNoise  
kaptcha.noise.color   验证码噪点颜色   默认为Color.BLACK  
kaptcha.obscurificator.impl   验证码样式引擎  默认为WaterRipple  
kaptcha.word.impl   验证码文本字符渲染   默认为DefaultWordRenderer  
kaptcha.background.impl   验证码背景生成器   默认为DefaultBackground  
kaptcha.background.clear.from   验证码背景颜色渐进   默认为Color.LIGHT_GRAY  
kaptcha.background.clear.to   验证码背景颜色渐进   默认为Color.WHITE  
kaptcha.image.width   验证码图片宽度  默认为200  
kaptcha.image.height  验证码图片高度  默认为50
kaptcha.session.key	session key	KAPTCHA_SESSION_KEY
kaptcha.session.date	session date	KAPTCHA_SESSION_DATE

```   
```

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" 
    xmlns="http://java.sun.com/xml/ns/javaee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
  
   <!-- kaptcha验证码配置 -->
    <servlet>
        <!-- 生成图片的Servlet -->
        <servlet-name>Kaptcha</servlet-name>
        <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
        
        <!-- 是否有边框-->
        <init-param>
            <param-name>kaptcha.border</param-name>
            <param-value>no</param-value>
        </init-param>    
        <!-- 字体颜色 -->
        <init-param>
            <param-name>kaptcha.textproducer.font.color</param-name>
            <param-value>red</param-value>
        </init-param>
        <!-- 图片宽度 -->
        <init-param>
            <param-name>kaptcha.image.width</param-name>
            <param-value>135</param-value>
        </init-param>
        <!-- 使用哪些字符生成验证码 -->
        <init-param>
            <param-name>kaptcha.textproducer.char.string</param-name>
            <param-value>ACDEFHKPRSTWX345679</param-value>
        </init-param>
        <!-- 图片高度 -->
        <init-param>
            <param-name>kaptcha.image.height</param-name>
            <param-value>50</param-value>
        </init-param>
        <!-- 字体大小 -->
        <init-param>
            <param-name>kaptcha.textproducer.font.size</param-name>
            <param-value>43</param-value>
        </init-param>
        <!-- 干扰线的颜色 -->
        <init-param>
            <param-name>kaptcha.noise.color</param-name>
            <param-value>black</param-value>
        </init-param>
        <!-- 字符个数 -->
        <init-param>
            <param-name>kaptcha.textproducer.char.length</param-name>
            <param-value>4</param-value>
        </init-param>
        <!-- 使用哪些字体 -->
        <init-param>
            <param-name>kaptcha.textproducer.font.names</param-name>
            <param-value>Arial</param-value>
        </init-param>        
    </servlet>
    <!-- 映射的url -->
    <servlet-mapping>
        <servlet-name>Kaptcha</servlet-name>
        <url-pattern>/Kaptcha.jpg</url-pattern>
    </servlet-mapping>
</web-app>

```
参考：http://stone02111.iteye.com/blog/1688195
http://blog.csdn.net/rambo_china/article/details/7720181
http://www.tuicool.com/articles/Fviuqe
http://www.cnblogs.com/xdp-gacl/p/4221848.html