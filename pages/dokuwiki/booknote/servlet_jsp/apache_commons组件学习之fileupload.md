title: apache_commons组件学习之fileupload 

#  apache_commons组件学习之fileupload 
FileUpload组件中定义了很多类，但是通常我们只需要与以下类打交道：
While this package provides the generic functionality for file uploads, these classes are not typically used directly. Instead, normal usage involves one of the provided extensions of FileUpload such as ServletFileUpload or PortletFileUpload, together with a factory for FileItem instances, such as DiskFileItemFactory.
  * 即:FileUpload接口、ServletFileUpload实现类：用于访问表单域或文件域。并设置监听器等。还可用于检测，配置等。处理上传。
  * FilleItem:读取相关信息，及执行具体写入读取操作。
  * DiskFileItemFactory：用于设定相关参数，同时用于构造ServletFileUpload.
  * Streams:用于流式API处理。
传统API处理时需要通过DiskFileItemFactory实例来构造ServletFileUpload对象，以此决定通过ServletFileUpload处理上传的文件的存储方式：在memory-or-disk（存储方式由DiskFIleItemFactory配置。）。
` 注意：FileUpload组件依赖于commons IO组件。 `
 The following is a brief example of typical usage in a servlet, storing the uploaded files on disk.
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
（可以参考这一段：
In the example above, the first file is loaded into memory as a String. Before calling the getString method, the data may have been in memory or on disk depending on its size. The second file we assume it will be large and therefore never explicitly load it into memory, though if it is less than 4096 bytes it will be in memory before it is written to its final location. When writing to the final location, if the data is larger than the threshold, an attempt is made to rename the temporary file to the given location. If it cannot be renamed, it is streamed to the new location.）
表单示例：
<form method="POST" enctype="multipart/form-data" action="fup.cgi">
  File to upload: <input type="file" name="upfile"><br/>
  Notes about the file: <input type="text" name="note"><br/>
  <br/>
  <input type="submit" value="Press"> to upload the file!
</form>
我们可以有两种方式进行：1.传统API来实现，方便；2.Stream API流式处理：性能更好。更快。
先介绍传统API方式:
其实我们在处理请求之前应该进行处理以确定其是否有文件域：通过如下方式：
```

// Check that we have a file upload request
boolean isMultipart = ServletFileUpload.isMultipartContent(request);
之后我们才能进行文件上传的解析。

```

几个操作点：

The simplest usage scenario is the following:
   Uploaded items should be retained in memory as long as they are reasonably small.
   Larger items should be written to a temporary file on disk.
   Very large upload requests should not be permitted.
   The built-in defaults for the maximum size of an item to be retained in memory, the maximum permitted size of an upload request, and the location of temporary files are acceptable.
// Create a factory for disk-based file items
DiskFileItemFactory factory = new DiskFileItemFactory();
// Configure a repository (to ensure a secure temp location is used)
ServletContext servletContext = this.getServletConfig().getServletContext();
//存至javax.servlet.context.tmpdir这个目录由容器自动配置的，所以是安全的。
 File repository = (File) servletContext.getAttribute("javax.servlet.context.tempdir");
factory.setRepository(repository);
// Create a new file upload handler
ServletFileUpload upload = new ServletFileUpload(factory);
// Parse the request
List<FileItem> items = upload.parseRequest(request);
你还可以更好地定制文件上传中的处理行为


// Create a factory for disk-based file items
DiskFileItemFactory factory = new DiskFileItemFactory();
// Set factory constraints
factory.setSizeThreshold(yourMaxTmpMemorySize);
factory.setRepository(yourTempDirectory);
// Create a new file upload handler
ServletFileUpload upload = new ServletFileUpload(factory);
// Set overall request size constraint
upload.setSizeMax(yourMaxRequestSize);
// Parse the request
List<FileItem> items = upload.parseRequest(request);
当然你也可以使用工厂类的构造器一并配置：
// Create a factory for disk-based file items
DiskFileItemFactory factory = new DiskFileItemFactory(yourMaxTmpMemorySize, yourTempDirectory);
注意：从表单普通的文件输入组件传上来的字符串也是当做FileItem的，所以我们需要再次确认所处理的FileItem是不是文件域。

// Process the uploaded items
Iterator<FileItem> iter = items.iterator();
while (iter.hasNext()) {
    FileItem item = iter.next();
    if (item.isFormField()) {//不是文件域，而是普通的表单域。
        processFormField(item);
    } else {
        processUploadedFile(item);//否则就是文件域，我们调用文件上传处理方法。
    }
}
对于普通的表单域我们一般需要知道其名称与值：通过：

// Process a regular form field
if (item.isFormField()) {
    String name = item.getFieldName();
    String value = item.getString();
    ...
}
对于文件域：
// Process a file upload
if (!item.isFormField()) {
    String fieldName = item.getFieldName();
    String fileName = item.getName();
    String contentType = item.getContentType();
    boolean isInMemory = item.isInMemory();
    long sizeInBytes = item.getSize();
    ...
}
将文件写入磁盘：示例：

// Process a file upload
if (writeToFile) {//writeToFile只是我们自己设置的标识。
    File uploadedFile = new File(...);
    item.write(uploadedFile);//写入文件
} else {
    InputStream uploadedStream = item.getInputStream();
    ...
    uploadedStream.close();
}
如果你需要处理数据可以这样：

// Process a file upload in memory
byte[] data = item.get();
如果上传的文件很大的话，我们需要监听上传情况，并提供进度条：


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
 fileupload.setProgressListener(progressListener);//通过ServletFileUpload对象注册监听器
但是以上监听太过频繁，所以我们需要修改一下：
/Create a progress listener
ProgressListener progressListener = new ProgressListener(){
   private long megaBytes = -1;
   public void update(long pBytesRead, long pContentLength, int pItems) {
       long mBytes = pBytesRead / 1000000;
       if (megaBytes == mBytes) {
           return;
       }
       megaBytes = mBytes;
       System.out.println("We are currently reading item " + pItems);
       if (pContentLength == -1) {
           System.out.println("So far, " + pBytesRead + " bytes have been read.");
       } else {
           System.out.println("So far, " + pBytesRead + " of " + pContentLength
                              + " bytes have been read.");
       }
   }
};
注意事项：一般浏览器item.getName()返回的是文件上传时的basename但是某些浏览器不同，如IE,opera等。
对于IE浏览器上传的不是基础文件名，而是路径名所以需要下面这样：
  String fileName = item.getName();
    if (fileName != null) {
        filename = FilenameUtils.getName(filename);
    }
自动清理临时文件的配置，一般自己产生的临时文件需要自己去清理，天经地义。
配置步骤：
1.配置web.xml:
<listener>
   <listener-class>
     org.apache.commons.fileupload.servlet.FileCleanerCleanup
   </listener-class>
 </listener>
2.将DiskFileItem的获取方式改为通过以下方法获取：
//配置追踪器。
public static DiskFileItemFactory getDiskFileItemFactory(ServletContext context,File repository) {
   FileCleaningTracker fileCleaningTracker
       = FileCleanerCleanup.getFileCleaningTracker(context);
   DiskFileItemFactory factory
       = new DiskFileItemFactory(DiskFileItemFactory.DEFAULT_SIZE_THRESHOLD,repository);
   factory.setFileCleaningTracker(fileCleaningTracker);
   return factory;
}
这样它就会自动删除不再使用的临时文件
临时文件清理的原理：  FileCleaningTracker会启动一个 reaper thread 即清理线程用于清理临时文件，
当不再需要清理时，这个线程需要通过Servlet Context中的监听器：org.apache.commons.fileupload.servlet.FileCleanerCleanup来终止这个线程 
2.StreamAPI流式处理：
流式处理为低内存消耗，快速处理的一中方式。它不用使用DiskFileItemFactory:
核心类Streams.
// Check that we have a file upload request
boolean isMultipart = ServletFileUpload.isMultipartContent(request);
// Create a new file upload handler直接构造无需使用DIskFileItemFactory。
ServletFileUpload upload = new ServletFileUpload();
// Parse the request获取FileItem的迭代器。
FileItemIterator iter = upload.getItemIterator(request);
while (iter.hasNext()) {
    FileItemStream item = iter.next();
    String name = item.getFieldName();
    InputStream stream = item.openStream();//打开获取文件流。
    if (item.isFormField()) {
        System.out.println("Form field " + name + " with value "
            + Streams.asString(stream) + " detected.");//通过Streams类进行处理。
//Streams除了可以将流转换为String外，还可以将流copy到输出流，并可指定是否关闭流，及临时缓存大小。
 } else {
        System.out.println("File field " + name + " with file name "
            + item.getName() + " detected.");
        // Process the input stream
        ...
    }
}