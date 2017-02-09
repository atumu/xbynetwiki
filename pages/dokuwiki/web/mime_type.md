title: mime_type 

# MIME Type 
 {".3gp", "video/3gpp"},  
{".apk", "application/vnd.android.package-archive"},  
{".asf", "video/x-ms-asf"},  
{".avi", "video/x-msvideo"},  
{".bin", "application/octet-stream"},  
{".bmp", "image/bmp"},  
{".c", "text/plain"},  
{".class", "application/octet-stream"},  
{".conf", "text/plain"},  
{".cpp", "text/plain"},  
{".doc", "application/msword"},  
{".exe", "application/octet-stream"},  
{".gif", "image/gif"},  
{".gtar", "application/x-gtar"},  
{".gz", "application/x-gzip"},  
{".h", "text/plain"},  
{".htm", "text/html"},  
{".html", "text/html"},  
{".jar", "application/java-archive"},  
{".java", "text/plain"},  
{".jpeg", "image/jpeg"},  
{".jpg", "image/jpeg"},  
{".js", "application/x-javascript"},  
{".log", "text/plain"},  
{".m3u", "audio/x-mpegurl"},  
{".m4a", "audio/mp4a-latm"},  
{".m4b", "audio/mp4a-latm"},  
{".m4p", "audio/mp4a-latm"},  
{".m4u", "video/vnd.mpegurl"},  
{".m4v", "video/x-m4v"},  
{".mov", "video/quicktime"},  
{".mp2", "audio/x-mpeg"},  
{".mp3", "audio/x-mpeg"},  
{".mp4", "video/mp4"},  
{".mpc", "application/vnd.mpohun.certificate"},  
{".mpe", "video/mpeg"},  
{".mpeg", "video/mpeg"},  
{".mpg", "video/mpeg"},  
{".mpg4", "video/mp4"},  
{".mpga", "audio/mpeg"},  
{".msg", "application/vnd.ms-outlook"},  
{".ogg", "audio/ogg"},  
{".pdf", "application/pdf"},  
{".png", "image/png"},  
{".pps", "application/vnd.ms-powerpoint"},  
{".ppt", "application/vnd.ms-powerpoint"},  
{".prop", "text/plain"},  
{".rar", "application/x-rar-compressed"},  
{".rc", "text/plain"},  
{".rmvb", "audio/x-pn-realaudio"},  
{".rtf", "application/rtf"},  
{".sh", "text/plain"},  
{".tar", "application/x-tar"},  
{".tgz", "application/x-compressed"},  
{".txt", "text/plain"},  
{".wav", "audio/x-wav"},  
{".wma", "audio/x-ms-wma"},  
{".wmv", "audio/x-ms-wmv"},  
{".wps", "application/vnd.ms-works"},  
{".xml", "text/xml"},  
{".xml", "text/plain"},  
{".z", "application/x-compress"},  
{".zip", "application/zip"},  
{"", "*/*"}  

##  一、MIME TYPE描述 

多用途互联网邮件扩展（MIME，Multipurpose Internet Mail Extensions）是一个互联网标准，它扩展了电子邮件标准，使其能够支持非ASCII字符、二进制格式附件等多种格式的邮件消息。
内容类型（Content-Type），这个头部领域用于指定消息的类型。一般以下面的形式出现。[type]/[subtype]
type有下面的形式。
Text：用于标准化地表示的文本信息，文本消息可以是多种字符集和或者多种格式的；
Multipart：用于连接消息体的多个部分构成一个消息，这些部分可以是不同类型的数据；
Application：用于传输应用程序数据或者二进制数据；
Message：用于包装一个E-mail消息；
Image：用于传输静态图片数据；
Audio：用于传输音频或者音声数据；
Video：用于传输动态影像数据，可以是与音频编辑在一起的视频数据格式。
subtype用于指定type的详细形式。content-type/subtype配对的集合和与此相关的参数，将随着时间而增长。为了确保这些值在一个有序而且公开的状态下开发，MIME使用Internet Assigned Numbers Authority (IANA)作为中心的注册机制来管理这些值。常用的subtype值如下所示：
text/plain（纯文本）
text/html（HTML文档）
application/xhtml+xml（XHTML文档）
image/gif（GIF图像）
image/jpeg（JPEG图像）【PHP中为：image/pjpeg】
image/png（PNG图像）【PHP中为：image/x-png】
video/mpeg（MPEG动画）
application/octet-stream（任意的二进制数据）
application/pdf（PDF文档）
application/msword（Microsoft Word文件）
message/rfc822（RFC 822形式）
multipart/alternative（HTML邮件的HTML形式和纯文本形式，相同内容使用不同形式表示）
application/x-www-form-urlencoded（使用HTTP的POST方法提交的表单）
multipart/form-data（同上，但主要用于表单提交时伴随文件上传的场合）