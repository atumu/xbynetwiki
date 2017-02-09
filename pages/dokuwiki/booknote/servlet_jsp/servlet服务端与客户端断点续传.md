title: servlet服务端与客户端断点续传 

#  Servlet服务端与客户端断点续传 
##  断点续传的原理 
其实断点续传的原理很简单，就是在 Http 的请求上和一般的下载有所不同而已。 
打个比方，浏览器请求服务器上的一个文时，所发出的请求如下： 
假设服务器域名为 wwww.sjtu.edu.cn，文件名为 down.zip。 
GET /down.zip HTTP/1.1 
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms- 
excel, application/msword, application/vnd.ms-powerpoint, */* 
Accept-Language: zh-cn 
Accept-Encoding: gzip, deflate 
User-Agent: Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0) 
Connection: Keep-Alive
服务器收到请求后，按要求寻找请求的文件，提取文件的信息，然后返回给浏览器，返回信息如下：
200 
` Content-Length=106786028 ` 
` Accept-Ranges=bytes ` 
Date=Mon, 30 Apr 2001 12:56:11 GMT 
ETag=W/"02ca57e173c11:95b" 
**Content-Type=application/octet-stream** 
Server=Microsoft-IIS/5.0 
Last-Modified=Mon, 30 Apr 2001 12:56:11 GMT
所谓断点续传，也就是要从文件已经下载的地方开始继续下载。所以在客户端浏览器传给 Web 服务器的时候要多加一条信息 -- 从哪里开始。 
下面是用自己编的一个"浏览器"来传递请求信息给 Web 服务器，要求从 2000070 字节开始。 
GET /down.zip HTTP/1.0 
User-Agent: NetFox 
` RANGE: bytes=2000070- ` 
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
仔细看一下就会发现多了一行 RANGE: bytes=2000070- 
这一行的意思就是告诉服务器 down.zip 这个文件从 2000070 字节开始传，前面的字节不用传了。 
服务器收到这个请求以后，返回的信息如下： 
` 206 `  206代表断点续传 
` Content-Length=106786028 ` 
` Content-Range=bytes 2000070-106786027/106786028 ` 
Date=Mon, 30 Apr 2001 12:55:20 GMT 
ETag=W/"02ca57e173c11:95b" 
**Content-Type=application/octet-stream** 
Server=Microsoft-IIS/5.0 
Last-Modified=Mon, 30 Apr 2001 12:55:20 GMT
和前面服务器返回的信息比较一下，就会发现增加了一行： 
Content-Range=bytes 2000070-106786027/106786028 
` 返回的代码也改为 206 了，而不再是 200 了。 `
知道了以上原理，就可以进行断点续传的编程了。
` 断点续传需要服务端的支持 `
##  服务端实现断点续传 
本文是 Java 服务器端支持 HTTP 断点续传的源代码，支持快车、迅雷。
```

//HTTP 断点续传 demo（客户端测试工具：快车、迅雷）
public class ArcSyncHttpDownloadServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	final static Log log = LogFactory.getLog(ArcSyncHttpDownloadServlet.class);
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		this.doPost(req, resp);
	}
	
	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response) {
			// TODO Auto-generated method stub
		File downloadFile = new File("D:\\222.mp4");
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

		try {
			resp.addHeader("Content-Disposition",
					"attachment; filename=\"" + new String(downloadFile.getName().getBytes(), "ISO-8859-1") + "\"");
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
}

```
##  客户端实现断点续传 
Java 实现断点续传的关键几点
(1) 用什么方法实现提交** RANGE: bytes=2000070-**。 
```

URL url = new URL("http://www.sjtu.edu.cn/down.zip"); 
HttpURLConnection httpConnection = (HttpURLConnection)url.openConnection(); 
// 设置 User-Agent 
httpConnection.setRequestProperty("User-Agent","NetFox"); 
// 设置断点续传的开始位置 
httpConnection.setRequestProperty("RANGE","bytes=2000070"); 
// 获得输入流 
InputStream input = httpConnection.getInputStream(); 
从输入流中取出的字节流就是 down.zip 文件从 2000070 开始的字节流。 大家看，其实断点续传用 Java 实现起来还是很简单的吧。 接下来要做的事就是怎么保存获得的流到文件中去了。
保存文件采用的方法。 我采用的是 IO 包中的 RandAccessFile 类。 操作相当简单，假设从 2000070 处开始保存文件，代码如下： 
RandomAccess oSavedFile = new RandomAccessFile("down.zip","rw"); 
long nPos = 2000070; 
// 定位文件指针到 nPos 位置 
oSavedFile.seek(nPos); 
byte[] b = new byte[1024]; 
int nRead; 
// 从输入流中读入字节流，然后写到文件中 
while((nRead=input.read(b,0,1024)) > 0) 
{ 
oSavedFile.write(b,0,nRead); 
}

```
怎么样，也很简单吧。 接下来要做的就是整合成一个完整的程序了。包括一系列的线程控制等等。


参考：
http://www.ibm.com/developerworks/cn/java/joy-down/
http://blog.csdn.net/defonds/article/details/7074352