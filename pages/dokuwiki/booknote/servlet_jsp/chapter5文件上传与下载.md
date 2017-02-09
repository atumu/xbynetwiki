title: chapter5文件上传与下载 

#  Chapter5文件上传与下载 
##  使用Servlet3上传 
**@MultipartConfig与javax.servlet.http.Part**
  * 1、使用注解@MultipartConfig将一个Servlet标识为支持文件上传。
  * 2、Servlet3.0将multipart/form-data的POST请求封装成Part，通过Part对上传的文件进行操作。

` 注意Part的getName返回的是表单域名，而非文件名，文件名需要自己手动获取 `
```

 <!-- 文件上传时必须要设置表单的enctype="multipart/form-data"-->
 <form action="${pageContext.request.contextPath}/UploadServlet"
     method="post" enctype="multipart/form-data">
      上传文件：
      <input type="file" name="file">
      <br>
      <input type="submit" value="上传">
 </form>
<!-- 文件上传时必须要设置表单的enctype="multipart/form-data"-->
 <form action="${pageContext.request.contextPath}/UploadServlet"
      method="post" enctype="multipart/form-data">
      上传文件：
      <input type="file" name="file1">
      <br>
     上传文件：
      <input type="file" name="file2">
      <br>
      <input type="submit" value="上传">
  </form>

```
```

//使用@WebServlet配置UploadServlet的访问路径
@WebServlet(name="UploadServlet",urlPatterns="/UploadServlet")
//使用注解@MultipartConfig将一个Servlet标识为支持文件上传
@MultipartConfig(  
        //location = "/upload",//文件存放临时路径，指定的目录必须存在，否则会抛异常。一般采用默认即可
        maxFileSize = 8388608,//最大上传文件大小,经测试应该是字节为单位 。如5M为5242880。限制单个文件大小
        fileSizeThreshold = 819200//设定一个溢出大小，当数据量大于该值时，内存中的内容将被写入文件。（specification中的解释的大概意思，不知道是不是指Buffer size），大小也是已字节单位  
        //maxRequestSize =  8*1024*1024*6 //针对该 multipart/form-data 请求的最大大小，默认值为 -1，表示没有限制。以字节为单位。考虑到多个文件同时上传。可以用这个参数进行限制
) 

public class UploadServlet extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
             request.setCharacterEncoding("utf-8");
            response.setCharacterEncoding("utf-8");
            response.setContentType("text/html;charset=utf-8");
            //存储路径
            String savePath = request.getServletContext().getRealPath("/WEB-INF/uploadFile");
            //获取上传的文件集合
            Collection<Part> parts = request.getParts();
            //上传单个文件
            if (parts.size()==1) {
                 //Servlet3.0将multipart/form-data的POST请求封装成Part，通过Part对上传的文件进行操作。
                //Part part = parts[0];//从上传的文件集合中获取Part对象
                Part part = request.getPart("file");//通过表单file控件(<input type="file" name="file">)的名字直接获取Part对象
                //Servlet3没有提供直接获取文件名的方法,需要从请求头中解析出来
                //获取请求头，请求头的格式：form-data; name="file"; filename="snmp4j--api.zip"
                String header = part.getHeader("content-disposition");
                //获取文件名
                String fileName = getFileName(header);
                //把文件写到指定路径
                part.write(savePath+File.separator+fileName);
            }else {
                //一次性上传多个文件
                for (Part part : parts) {//循环处理上传的文件
                    //获取请求头，请求头的格式：form-data; name="file"; filename="snmp4j--api.zip"
                    String header = part.getHeader("content-disposition");
                    //获取文件名
                    String fileName = getFileName(header);
                    //把文件写到指定路径
                    part.write(savePath+File.separator+fileName);
                }
            }
            PrintWriter out = response.getWriter();
            out.println("上传成功");
            out.flush();
            out.close();
    }

     /**
     * 根据请求头解析出文件名
     * 请求头的格式：火狐和google浏览器下：form-data; name="file"; filename="snmp4j--api.zip"
     *                 IE浏览器下：form-data; name="file"; filename="E:\snmp4j--api.zip"
     * @param header 请求头
     * @return 文件名
     */
    public String getFileName(String header) {
        /**
         * String[] tempArr1 = header.split(";");代码执行完之后，在不同的浏览器下，tempArr1数组里面的内容稍有区别
         * 火狐或者google浏览器下：tempArr1={form-data,name="file",filename="snmp4j--api.zip"}
         * IE浏览器下：tempArr1={form-data,name="file",filename="E:\snmp4j--api.zip"}
         */
        String[] tempArr1 = header.split(";");
        /**
         *火狐或者google浏览器下：tempArr2={filename,"snmp4j--api.zip"}
         *IE浏览器下：tempArr2={filename,"E:\snmp4j--api.zip"}
         */
        String[] tempArr2 = tempArr1[2].split("=");
      for(String str:tempArr2){
        if(str.trim().startsWith("filename")){
        //获取文件名
        String fileName = str.substring(str.lastIndexOf("=")+1).trim().replaceAll("\"", "");
          //防止乱码
        return new String(fileName.getBytes(),"UTF-8");
        }
      }
    }
    
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}

``` 
如果不想通过注解配置，可以在<servlet>标签中，添加一个<multipart-config>标签，在该标签中可以使用<location>、<file-size-threshold>、<max-file-size>和<max-request-size>标签。

##  使用Apache Commons-FileUpload方式上传 
需要添加第三方提供的jar包
1) commons-fileupload 
2) commons-io
文件上传的表单提交方式必须是POST方式，
编码类型：enctype="multipart/form-data"，默认是 application/x-www-form-urlencoded
比如：` <form action="FileUpLoad" enctype="multipart/form-data" method="post"> `

即:FileUpload接口、**ServletFileUpload**实现类：用于访问表单域或文件域。并设置监听器等。还可用于检测，配置等。处理上传。
**FilleItem**:读取相关信息，及执行具体写入读取操作。
**DiskFileItemFactory**：用于设定相关参数，同时用于构造ServletFileUpload.
**Streams**:用于流式API处理。
传统API处理时需要通过DiskFileItemFactory实例来构造ServletFileUpload对象，以此决定通过ServletFileUpload处理上传的文件的存储方式：在memory-or-disk（存储方式由DiskFIleItemFactory配置。）。
` 注意：FileUpload组件依赖于commons IO组件。 `
**其实我们在处理请求之前应该进行处理以确定其是否有文件域：通过如下方式：**
` boolean isMultipart = ServletFileUpload.isMultipartContent(request); `
之后我们才能进行文件上传的解析。
**存至javax.servlet.context.tmpdir这个目录由容器自动配置的，所以是安全的。**
''DiskFileItemFactory factory = new DiskFileItemFactory();
Configure a repository (to ensure a secure temp location is used)
ServletContext servletContext = this.getServletConfig().getServletContext();
File repository = (File) servletContext.getAttribute(“javax.servlet.context.tempdir”);
factory.setRepository(repository);''
**设置允许上传的最大大小。**
''ServletFileUpload upload = new ServletFileUpload(factory);
 // maximum size before a FileUploadException will be thrown
 upload.setSizeMax(1000000);//这里才是设置允许上传的最大大小。''
**设置内存TMP上限**
` factory.setSizeThreshold(yourMaxTmpMemorySize); `
**再次确认所处理的FileItem是不是文件域**
**注意：从表单普通的文件输入组件传上来的字符串也是当做FileItem的**，所以我们需要**再次确认所处理的FileItem是不是文件域。**
```

Iterator<FileItem> iter = items.iterator();
while (iter.hasNext()) {
FileItem item = iter.next();
if (item.isFormField()) {不是文件域，而是普通的表单域。
   processFormField(item);
} else {
  processUploadedFile(item);//否则就是文件域，我们调用文件上传处理方法。
}

```
**自动清理临时文件的配置**
一般自己产生的临时文件需要自己去清理，天经地义。
配置步骤：
```

1.配置web.xml:
<listener>
<listener-class>
org.apache.commons.fileupload.servlet.FileCleanerCleanup
</listener-class>
</listener>

```
2.将DiskFileItem的获取方式改为通过以下方法获取：
配置追踪器。
```

public static DiskFileItemFactory getDiskFileItemFactory(ServletContext context,File repository) {
 FileCleaningTracker fileCleaningTracker
     = FileCleanerCleanup.getFileCleaningTracker(context);
 DiskFileItemFactory factory
     = new DiskFileItemFactory(DiskFileItemFactory.DEFAULT_SIZE_THRESHOLD,repository);
 factory.setFileCleaningTracker(fileCleaningTracker);
 return factory;
}

```
**监听上传情况**
''ProgressListener progressListener = new ProgressListener(){...}
fileupload.setProgressListener(progressListener);通过ServletFileUpload对象注册监听器''
```

//Create a progress listener
ProgressListener progressListener = new ProgressListener(){
   public void update(long pBytesRead, long pContentLength, int pItems) {
       System.out.println("We are currently reading item " + pItems);
       if (pContentLength == -1) {
           System.out.println("So far, " + pBytesRead + " bytes have been read.");
       } else {
           System.out.println("So far, " + pBytesRead + " of " + pContentLength
                              + " bytes have been read.");
       }
   }
};
upload.setProgressListener(progressListener);

```
**2.StreamAPI流式处理：**
流式处理为低内存消耗，快速处理的一中方式。它不用使用DiskFileItemFactory:
核心类**Streams**.
```

public void doPost(HttpServletRequest req, HttpServletResponse res) {
   DiskFileItemFactory factory = new DiskFileItemFactory();
   // maximum size that will be stored in memory,看清楚哦，这是不是指限制文件上传的大小，
   //而是指文件小于这个值时直接放在内存中，而文件大小大于这个值的文件会被暂存在临时仓库，也就是 factory.setRepository()这里。
    factory.setSizeThreshold(4096);
   // the location for saving data that is larger than getSizeThreshold()，这只是一个临时文件存放仓库哦。注意。
   factory.setRepository(new File("/tmp"));
   ServletFileUpload upload = new ServletFileUpload(factory);
   // maximum size before a FileUploadException will be thrown
   upload.setSizeMax(1000000);//这里才是设置允许上传的最大大小。
   List<FileItem> fileItems = upload.parseRequest(req);//可能会有多个文件。所以返回列表
   // assume we know there are two files. The first file is a small
   // text file, the second is unknown and is written to a file on
   // the server
   Iterator i = fileItems.iterator();
   String comment = ((FileItem)i.next()).getString();
   FileItem fi = (FileItem)i.next();
   // filename on the client
   String fileName = fi.getName();
   // save comment and filename to database
   ...
   // write the file
   fi.write(new File("/www/uploads/", fileName));
 }

```
流式API:
```

// Check that we have a file upload request
boolean isMultipart = ServletFileUpload.isMultipartContent(request);
// Create a new file upload handler
ServletFileUpload upload = new ServletFileUpload();

// Parse the request
FileItemIterator iter = upload.getItemIterator(request);
while (iter.hasNext()) {
    FileItemStream item = iter.next();
    String name = item.getFieldName();
    InputStream stream = item.openStream();
    if (item.isFormField()) {
        System.out.println("Form field " + name + " with value "
            + Streams.asString(stream) + " detected.");
    } else {
        System.out.println("File field " + name + " with file name "
            + item.getName() + " detected.");
        // Process the input stream
        ...
    }
}

```
###  举例： 
1.fileupload.jsp
```

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">  
<html>  
  <head>  
    <base href="<%=basePath%>">  
      
    <title>My JSP 'fileupload.jsp' starting page</title>  
      
    <meta http-equiv="pragma" content="no-cache">  
    <meta http-equiv="cache-control" content="no-cache">  
    <meta http-equiv="expires" content="0">      
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">  
    <meta http-equiv="description" content="This is my page">  
    <!-- 
    <link rel="stylesheet" type="text/css" href="styles.css"> 
    -->  
  
  </head>  
    
  <body>  
     <!-- enctype 默认是 application/x-www-form-urlencoded -->  
     <form action="FileUpLoad" enctype="multipart/form-data" method="post" >  
          
               用户名：<input type="text" name="usename"> <br/>  
               上传文件：<input type="file" name="file1"><br/>  
              上传文件： <input type="file" name="file2"><br/>  
              <input type="submit" value="提交"/>  
       
     </form>  
       
       
       
  </body>  
</html> 

```
2.实际处理文件上传的 FileUpLoad.java
```

import org.apache.commons.fileupload.FileItem;  
import org.apache.commons.fileupload.FileUploadException;  
import org.apache.commons.fileupload.disk.DiskFileItemFactory;  
import org.apache.commons.fileupload.servlet.ServletFileUpload;  
  
/** 
 *  
 * @author Administrator 
 * 文件上传 
 * 具体步骤： 
 * 1）获得磁盘文件条目工厂 DiskFileItemFactory 要导包 
 * 2） 利用 request 获取 真实路径 ，供临时文件存储，和 最终文件存储 ，这两个存储位置可不同，也可相同 
 * 3）对 DiskFileItemFactory 对象设置一些 属性 
 * 4）高水平的API文件上传处理  ServletFileUpload upload = new ServletFileUpload(factory); 
 * 目的是调用 parseRequest（request）方法  获得 FileItem 集合list ， 
 *      
 * 5）在 FileItem 对象中 获取信息，   遍历， 判断 表单提交过来的信息 是否是 普通文本信息  另做处理 
 * 6） 
 *    第一种. 用第三方 提供的  item.write( new File(path,filename) );  直接写到磁盘上 
 *    第二种. 手动处理   
 * 
 */  
public class FileUpLoad extends HttpServlet {  
  
    public void doPost(HttpServletRequest request, HttpServletResponse response)  
            throws ServletException, IOException {  
          
        request.setCharacterEncoding("utf-8");  //设置编码  
          
        //获得磁盘文件条目工厂  
        DiskFileItemFactory factory = new DiskFileItemFactory();  
        //获取文件需要上传到的路径  
        String path = request.getRealPath("/upload");  
          
        //如果没以下两行设置的话，上传大的 文件 会占用 很多内存，  
        //设置暂时存放的 存储室 , 这个存储室，可以和 最终存储文件 的目录不同  
        /** 
         * 原理 它是先存到 暂时存储室，然后在真正写到 对应目录的硬盘上，  
         * 按理来说 当上传一个文件时，其实是上传了两份，第一个是以 .tem 格式的  
         * 然后再将其真正写到 对应目录的硬盘上 
         */  
        factory.setRepository(new File(path));  
        //设置 缓存的大小，当上传文件的容量超过该缓存时，直接放到 暂时存储室  
        factory.setSizeThreshold(1024*1024) ;  
          
        //高水平的API文件上传处理  
        ServletFileUpload upload = new ServletFileUpload(factory);  
          
          
        try {  
            //可以上传多个文件  
            List<FileItem> list = (List<FileItem>)upload.parseRequest(request);  
              
            for(FileItem item : list)  
            {  
                //获取表单的属性名字  
                String name = item.getFieldName();  
                  
                //如果获取的 表单信息是普通的 文本 信息  
                if(item.isFormField())  
                {                     
                    //获取用户具体输入的字符串 ，名字起得挺好，因为表单提交过来的是 字符串类型的  
                    String value = item.getString() ;  
                      
                    request.setAttribute(name, value);  
                }  
                //对传入的非 简单的字符串进行处理 ，比如说二进制的 图片，电影这些  
                else  
                {  
                    /** 
                     * 以下三步，主要获取 上传文件的名字 
                     */  
                    //获取路径名  
                    String value = item.getName() ;  
                    //索引到最后一个反斜杠  
                    int start = value.lastIndexOf("\\");  
                    //截取 上传文件的 字符串名字，加1是 去掉反斜杠，  
                    String filename = value.substring(start+1);  
                      
                    request.setAttribute(name, filename);  
                      
                    //真正写到磁盘上  
                    //它抛出的异常 用exception 捕捉  
                      
                    //item.write( new File(path,filename) );//第三方提供的  
                      
                    //手动写的  
                    OutputStream out = new FileOutputStream(new File(path,filename));  
                      
                    InputStream in = item.getInputStream() ;  
                      
                    int length = 0 ;  
                    byte [] buf = new byte[1024] ;  
                      
                    System.out.println("获取上传文件的总共的容量："+item.getSize());  
  
                    // in.read(buf) 每次读到的数据存放在   buf 数组中  
                    while( (length = in.read(buf) ) != -1)  
                    {  
                        //在   buf 数组中 取出数据 写到 （输出流）磁盘上  
                        out.write(buf, 0, length);  
                          
                    }  
                      
                    in.close();  
                    out.close();  
                }  
            }  
              
              
              
        } catch (FileUploadException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        catch (Exception e) {  
            // TODO Auto-generated catch block  
              
            //e.printStackTrace();  
        }  
          
          
        request.getRequestDispatcher("filedemo.jsp").forward(request, response);  
          
  
    }  
  
}  

```
System.out.println("获取上传文件的总共的容量："+item.getSize()); 
3.filedemo.jsp
```

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">  
<html>  
  <head>  
    <base href="<%=basePath%>">  
      
    <title>My JSP 'filedemo.jsp' starting page</title>  
      
    <meta http-equiv="pragma" content="no-cache">  
    <meta http-equiv="cache-control" content="no-cache">  
    <meta http-equiv="expires" content="0">      
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">  
    <meta http-equiv="description" content="This is my page">  
    <!-- 
    <link rel="stylesheet" type="text/css" href="styles.css"> 
    -->  
  
  </head>  
    
  <body>  
      
    用户名：${requestScope.usename } <br/>  
    文件：${requestScope.file1 }<br/>  
    ${requestScope.file2 }<br/>  
    <!-- 把上传的图片显示出来 -->  
    <img alt="go" src="upload/<%=(String)request.getAttribute("file1")%> " />  
      
      
      
  </body>  
</html>  

```
##  Servlet文件下载 

文件下载本质是**流的复制输出**。比较重要的两点是文件名的**乱码问题和断点续传问题**。
''response.setContentType("application/Octet-stream;charset=UTF-8");  
response.setHeader("Content-Disposition", "attachment; filename="+filename); ''
响应设置的头Content-Disposition,将强制浏览器询问客户是保存还是下载文件。设置的内容类型为通用的，二进制内容类型。当然更恰当的方式是设置具体的MIME类型。
```

//File dfiled = new File(getServletContext().getRealPath("/down")+"shell.rar");  
 //你还需要判断文件是否存在等
//得到输入流  
FileInputStream in = new FileInputStream(dfile);  
//获取文件名字，并对其进行编码，如果不这样会出现中文乱码。至少用这种方式我暂时还没有出现过乱码  
String filename = dfile.getName();  
filename = new String(filename.getBytes(),"ISO-8859-1");  
  
//通过response对象获得输出流  
OutputStream out = response.getOutputStream();  
//  
response.setContentType("Application/Octet-stream;charset=utf-8");  
// 下载文件的名字通过这里设置  
response.addHeader("Content-Disposition", "attachment; filename="+filename);  
//headers.setContentDispositionFormData("attachment", URLEncoder.encode(entity.getFileName(), "UTF-8"));

   // 下面两句代码功能应该是一样的，都写上去会有异常，说重复设置响应头。  
// 但是只写第二句的时候用迅雷无法下载。而只写第一句文件可以下载，麻烦知道的朋友可以告诉我一下  
response.setContentLength((int) dfile.length());  
//response.addHeader("Content-Length::", dfile.length() + "");  
  
// 下面是一个普通的流的复制 。。。忽略 .这样可以防止内存问题
byte[] bs =new byte[1024];  
int len = 0;  
while((len =in.read(bs))!=-1){  
    out.write(bs);  
}  
// 最后是流的关闭。  
out.close();  
in.close();  

```
###  使用java nio方式实现下载 
```

 @Override
    public byte[] getFileByte() {
        File file = new File(filePath);
        byte[] result = null;
        FileChannel fc = null;
        try {
            fc = new RandomAccessFile(file, "r").getChannel();
            MappedByteBuffer byteBuffer = fc.map(MapMode.READ_ONLY, 0, fc.size()).load();
            result = new byte[(int) fc.size()];
            if (byteBuffer.remaining() > 0) {
                byteBuffer.get(result, 0, byteBuffer.remaining());
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fc.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return result;
    }

```
##  断点下载 
```

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.OutputStream;
import java.io.RandomAccessFile;
import java.lang.reflect.Method;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.security.AccessController;
import java.security.PrivilegedAction;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.tika.io.IOUtils;

import com.css.sword.cms.util.StringUtil;
import com.css.sword.utils.logger.SwordLogUtils;

public class DownloadHelp {
	public static void download(HttpServletRequest req,HttpServletResponse resp,String fileName,File downloadFile){
//		File downloadFile = new File("D:\\222.mp4");
		long fileLength = downloadFile.length();// 记录文件大小
		long pastLength = 0;// 记录已下载大小
		int rangeSwitch = 0; // 下载状态标记位，0-从头下载；1-从某字节开始下载（bytes=27000-）;2-从某字节到某某字节结束的下载（bytes=27000-39000）
		long toLength = 0;// 记录客户端需要下载字节的末端偏移量（比如bytes=27000-39000，则这个值是为39000）
		long contentLength = 0;// 客户端请求字节量
		String rangeBytes = "";// 用于记录客户端传来的形如”bytes=27000-“或者“bytes=27000-39000”的内容
		RandomAccessFile raf = null;
		OutputStream os = null;
		BufferedOutputStream out = null;// 缓冲
		byte bytes[] = new byte[1024];// 暂存容器
		if (StringUtil.checkNotEmpty(req.getHeader("Range"))) {
			// 若客户端传来Range头，说明客户端之前下载了一部分，设置206状态码
			resp.setStatus(HttpServletResponse.SC_PARTIAL_CONTENT);
			log.info("客户端支持断点续传Range:" + req.getHeader("Range"));
			rangeBytes = req.getHeader("Range").replaceAll("bytes=", "");
			if (rangeBytes.indexOf("-") == rangeBytes.length() - 1) { // rangeSwitch状态为1.（bytes=27000-）
				rangeSwitch = 1;
				pastLength = Long.parseLong(rangeBytes.substring(0, rangeBytes.indexOf('-')).trim());
				contentLength = fileLength - pastLength + 1;
			} else {// rangeSwitch状态为2.（bytes=27000-39000）
				rangeSwitch = 2;
				String tmpStart = rangeBytes.substring(0, rangeBytes.indexOf('-')).trim();
				String tmpEnd = rangeBytes.substring(rangeBytes.indexOf('-') + 1, rangeBytes.length()).trim();
				pastLength = Long.parseLong(tmpStart);
				toLength = Long.parseLong(tmpEnd);
				contentLength = toLength - pastLength + 1;
			}
		} else { // 否则从头开始下载，客户端不支持断点续载
			contentLength = fileLength; // 200,无需显式设置
		}
		resp.reset();
		resp.setHeader("Accept-Ranges", "bytes");
		if (pastLength != 0) { // 不是从头开始下载
			log.info("文件断点下载");
			switch (rangeSwitch) {
			case 1: {
				// 响应的格式是: Content-Range: bytes [文件块的开始字节]-[文件的总大小 - 1]/[文件的总大小]
				String contentRange = new StringBuilder("bytes ").append(new Long(pastLength).toString()).append("-")
						.append(new Long(fileLength - 1).toString()).append("/").append(new Long(fileLength).toString())
						.toString();
				resp.setHeader("Content-Range", contentRange);
				break;
			}
			case 2: {
				String contentRange = rangeBytes + "/" + new Long(fileLength).toString();
				resp.setHeader("Content-Range", contentRange);
				break;
			}
			default: {
				break;
			}
			}
		} else {
			log.info("文件从头下载");
		}
		
		MappedByteBuffer mapBuf=null;
		try {
			resp.addHeader("Content-Disposition",
					"attachment; filename=\"" + new String(fileName.getBytes(), "ISO-8859-1") + "\"");
			resp.setContentType("application/octet-stream");
//			resp.addHeader("Content-Length", String.valueOf(contentLength));
			resp.setContentLength((int)contentLength);;
			os = resp.getOutputStream();
			out = new BufferedOutputStream(os);
			raf = new RandomAccessFile(downloadFile, "r");
			mapBuf=raf.getChannel().map(FileChannel.MapMode.READ_ONLY, pastLength,downloadFile.length()-pastLength);
			while(mapBuf.hasRemaining()){
				out.write(mapBuf.get());
			}
//			try {
//				int n = 0;
//				while ((n = raf.read(bytes, 0, 1024)) != -1) {
//					out.write(bytes, 0, n);
//				}
//				out.flush();
//				os.flush();
//				
//			} catch (IOException e) {
//				e.printStackTrace();
//				log.error("文件下载出现异常{}",e);
//			}
		} catch (Exception e) {
			log.error("文件下载出现异常{}",e);
		} finally {
			IOUtils.closeQuietly(out);
			if (raf != null) {
				try {
					raf.close();
				} catch (IOException e) {
					log.error(e.getMessage(), e);
				}
			}
			unmap(mapBuf);
		}
	}
	@SuppressWarnings("unchecked")
	private static void unmap(final MappedByteBuffer byteBuffer){
		if(byteBuffer==null) return;
		AccessController.doPrivileged(new PrivilegedAction() {  
			  public Object run() {  
			    try {  
			      Method getCleanerMethod = byteBuffer.getClass().getMethod("cleaner", new Class[0]);  
			      getCleanerMethod.setAccessible(true);  
			      sun.misc.Cleaner cleaner = (sun.misc.Cleaner)   
			      getCleanerMethod.invoke(byteBuffer, new Object[0]);  
			      cleaner.clean();  
			      log.info("MappedByteBuffer释放完毕");
			    } catch (Exception e) {  
			      e.printStackTrace();
			      log.warn("内存释放异常{}",e);
			    }  
			    return null;  
			  }  
			});
	}
}


```
##  上传与下载的乱码或中文不显示问题解决： 
###  JSP或Servlet上传文件中文名乱码解决： 

```

String fileN=item.getName();
//String fileName=new String(fileN.getBytes("ISO-8859-1"),"utf-8");
String fileName=new String(fileN.getBytes(),"utf-8");//对提取出来的文件名进行UTF-8编码，

```
以上可以解决上传文件名乱码和浏览器显示以及存入数据库都显示正常。
###  解决下载文件时中文不显示的问题 
```

response.setContentType("application/x-msdownload");
//response.setContentType("application/octet-stream");
response.setHeader("Content-Disposition", "attachment;filename="+new String(fileName.getBytes("utf-8"),"ISO-8859-1"));//或者使用URL编码：URLEncoder.encode(filename, "UTF-8")

```
原因分析，由于HTTP头部的默认编码为ISO-8859-1而我们上传文件于下载文件过程中，提取到的文件名都要通过HTTP头部。
所以我们需要在上传的时候对提取到的文件名进行转码为UTF-8，然后在下载时我们也要进行反向转码为ISO-8859-1.
参考：
http://blog.csdn.net/xhmlwaf/article/details/8280000
http://www.cnblogs.com/xdp-gacl/p/4224960.html
http://pisces-java.iteye.com/blog/723125
http://www.blogjava.net/yongboy/archive/2011/01/15/346202.html