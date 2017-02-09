title: springmvc之文件上传与下载 

#  springmvc之文件上传与下载 
可供参考的资料：
http://blog.csdn.net/swingpyzf/article/details/20230865
#  文件上传 
SpringMVC支持两种方式的文件上传：
  * 方式一：采用Apache commons Fileupload开源库：依赖commons-fileupload,commons-io
  * 方式二：采用Servlet3.0自带的文件上传功能。

##  客户端编程 
```

<form action="" enctype="multipart/form-data" method="post">
	<input type="file" name="fieldName" multiple/>
	<input type="file" name="fieldName" />
</form>

```
##  SpringMVC的MultipartFile接口 
主要方法：
  * getBytes(),以字节数组的形式返回文件的内容
  * String getContentType()/ /获取文件MIME类型
  * InputStream getInputStream()/ /获取文件流
  * String getName() / /获取表单中文件组件的名字，非文件名
  * String ` getOriginalFilename() ` / /获取上传文件的原名（不带路径的那种）。这才是文件名
  * long getSize()  / /获取文件的字节大小，单位byte
  * boolean isEmpty() / /是否为空
  * void transferTo(File dest) / /将上传的文件保存到一个目标文件下。

##  使用Commons FileUpload方式上传文件 
依赖commons-fileupload,commons-io
```

<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.3.1</version>
</dependency>

```
首先，在SpringMVC配置文件中配置` multipartResolver ` bean
```

<!-- 配置MultipartResolver 用于文件上传 使用spring的CommosMultipartResolver -->  
    <beans:bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"  
        p:defaultEncoding="UTF-8"  
        p:maxUploadSize="5400000"  
        p:uploadTempDir="fileUpload/temp"  
     >  
    </beans:bean>

```
其中属性详解：
defaultEncoding="UTF-8" 是请求的编码格式，默认为iso-8859-1
maxUploadSize="5400000" 是上传文件的大小，单位为字节
uploadTempDir="fileUpload/temp" 为上传文件的临时路径
**第二步：编写controller**
```

 @RequestMapping("/fileUpload")  
    public String fileUpload(@RequestParam("file") MultipartFile file) {  
        // 判断文件是否为空  
        if (!file.isEmpty()) {  
            try {  
                // 文件保存路径  
                String filePath = request.getSession().getServletContext().getRealPath("/") + "upload/"  
                        + file.getOriginalFilename();
              // File file = new File(servletRequest.getServletContext().getRealPath("/file"), fileName);
              // File file = new File(servletRequest.getServletContext().getRealPath("/WEB-INF/file"),file.getOriginalFilename());
                // 转存文件  
                file.transferTo(new File(filePath));  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
        // 重定向  
        return "redirect:/list.html";  
    }  

```
###  多文件上传 
```

    private boolean saveFile(MultipartFile file) {  
        // 判断文件是否为空  
        if (!file.isEmpty()) {  
            try {  
                // 文件保存路径  
                String filePath = request.getSession().getServletContext().getRealPath("/") + "upload/"  
                        + file.getOriginalFilename();  
                // 转存文件  
                file.transferTo(new File(filePath));  
                return true;  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
        return false;  
    }  
 MultipartFile[]为数组形式。
    @RequestMapping("filesUpload")  
    public String filesUpload(@RequestParam("files") MultipartFile[] files) {  
        //判断file数组不能为空并且长度大于0  
        if(files!=null&&files.length>0){  
            //循环获取file数组中得文件  
            for(int i = 0;i<files.length;i++){  
                MultipartFile file = files[i];  
                //保存文件  
                saveFile(file);  
            }  
        }  
        // 重定向  
        return "redirect:/list.html";  
    } 

```
##  使用Servlet3方式上传文件 
步骤一：编辑web.xml添加multipart-config支持
```

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
	    <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/config/springmvc-config.xml</param-value>
		</init-param>
        <load-on-startup>1</load-on-startup>
		<multipart-config>
		    <max-file-size>20848820</max-file-size>
		    <max-request-size>418018841</max-request-size>
		    <file-size-threshold>1048576</file-size-threshold>
		</multipart-config>            
	</servlet>

```
步骤二：编辑SpringMVC的配置文件添加另一个multipartResolver bean:StandardServletMultipartResolver
```

	<bean id="multipartResolver"
	    class="org.springframework.web.multipart.support.StandardServletMultipartResolver">
	</bean>

```
其余步骤和方式一是一样的。
#  文件下载 
**需要对控制器完成以下工作：**
  * 请求处理方法返回值为void，方法参数中添加HttpServletResponse
  * 响应中Content-Type为mime类型或者application/octet-stream
  * 添加一个HTTP头Content-Disposition,值为attachment; filename=fileName
代码示例：
```

fis = new FileInputStream(file);
                bis = new BufferedInputStream(fis);
                OutputStream os = response.getOutputStream();
                int i = bis.read(buffer);
                while (i != -1) {
                    os.write(buffer, 0, i);
                    i = bis.read(buffer);
                }

```
###  范例1：对非登录用户隐藏下载： 

检索HttpSession中是否有自定义的属性logId来检测用户是否已成功登陆。
```

@Controller

public class ResourceController {
	
	private static final Log logger = LogFactory.getLog(ResourceController.class);
	
	@RequestMapping(value="/login")
	public String login(@ModelAttribute Login login, HttpSession session, Model model) {
	    model.addAttribute("login", new Login());
	    if ("paul".equals(login.getUserName()) &&
	            "secret".equals(login.getPassword())) {
	        session.setAttribute("loggedIn", Boolean.TRUE);
            return "Main";
	    } else {
	        return "LoginForm";
	    }
	}

	@RequestMapping(value="/resource_download")
	public String downloadResource(HttpSession session, HttpServletRequest request,
	        HttpServletResponse response) {
        if (session == null || 
                session.getAttribute("loggedIn") == null) {
            return "LoginForm";
        }
        String dataDirectory = request.
                getServletContext().getRealPath("/WEB-INF/data");
        File file = new File(dataDirectory, "secret.pdf");
        if (file.exists()) {
            response.setContentType("application/pdf");
            response.addHeader("Content-Disposition", 
                    "attachment; filename=secret.pdf");
            byte[] buffer = new byte[1024];
            FileInputStream fis = null;
            BufferedInputStream bis = null;
            // if using Java 7, use try-with-resources
            try {
                fis = new FileInputStream(file);
                bis = new BufferedInputStream(fis);
                OutputStream os = response.getOutputStream();
                int i = bis.read(buffer);
                while (i != -1) {
                    os.write(buffer, 0, i);
                    i = bis.read(buffer);
                }
            } catch (IOException ex) {
                // do something, 
                // probably forward to an Error page
            } finally {
                if (bis != null) {
                    try {
                        bis.close();
                    } catch (IOException e) {
                    }
                }
                if (fis != null) {
                    try {
                        fis.close();
                    } catch (IOException e) {
                    }
                }
            }
        }
        return null;
	}
	
}


```
###  范例2：防止交叉引用，防盗链 

通过检测` referer `头来进行判断
```

@Controller
public class ImageController {
	
	private static final Log logger = LogFactory.getLog(ImageController.class);
	
	@RequestMapping(value="/image_get/{id}", method = RequestMethod.GET)
	public void getImage(@PathVariable String id, 
	        HttpServletRequest request, 
	        HttpServletResponse response, 
	        @RequestHeader String referer) {
        if (referer != null) {
            String imageDirectory = request.getServletContext().
                    getRealPath("/WEB-INF/image");
            File file = new File(imageDirectory, 
                    id + ".jpg");
            if (file.exists()) {
                response.setContentType("image/jpg");
                byte[] buffer = new byte[1024];
                FileInputStream fis = null;
                BufferedInputStream bis = null;
                // if you're using Java 7, use try-with-resources
                try {
                    fis = new FileInputStream(file);
                    bis = new BufferedInputStream(fis);
                    OutputStream os = response.getOutputStream();
                    int i = bis.read(buffer);
                    while (i != -1) {
                        os.write(buffer, 0, i);
                        i = bis.read(buffer);
                    }
                } catch (IOException ex) {
                    // do something here
                } finally {
                    if (bis != null) {
                        try {
                            bis.close();
                        } catch (IOException e) {
                            
                        }
                    }
                    if (fis != null) {
                        try {
                            fis.close();
                        } catch (IOException e) {
                            
                        }
                    }
                }
            }
        }
	}
}

```
