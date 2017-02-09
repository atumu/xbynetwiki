title: linux命令之curl与wget 

#  Linux命令之curl与wget 
##  wget 
wget是一个下载文件的工具，它用在命令行下。
  * wget支持HTTP，HTTPS和FTP协议，可以使用HTTP代理。
  * wget可以在用户退出系统的之后在后台执行。这意味这你可以登录系统，启动一个wget下载任务，然后退出系统，wget将在后台执行直到任务完成
  * wget 可以跟踪HTML页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构。这又常被称作”递归下载”。在递归下载的时候，wget 遵循Robot Exclusion标准(/robots.txt). wget可以在下载的同时，将链接转换成指向本地文件，以方便离线浏览。
  * wget 非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性.如果是由于网络的原因下载失败，wget会不断的尝试，直到整个文件下载完毕。如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。

----

wget [参数] [URL地址]

启动参数：
-V, –version 显示wget的版本后退出
-h, –help 打印语法帮助
**-b, –background 启动后转入后台执行**
-e, –execute=COMMAND 执行`.wgetrc’格式的命令，wgetrc格式参见/etc/wgetrc或~/.wgetrc

----

记录和输入文件参数：
-o, –output-file=FILE 把记录写到FILE文件中
-a, –append-output=FILE 把记录追加到FILE文件中
-d, –debug 打印调试输出
-q, –quiet 安静模式(没有输出)
-v, –verbose 冗长模式(这是缺省设置)
-nv, –non-verbose 关掉冗长模式，但不是安静模式
-i, –input-file=FILE 下载在FILE文件中出现的URLs
-F, –force-html 把输入文件当作HTML格式文件对待
-B, –base=URL 将URL作为在-F -i参数指定的文件中出现的相对链接的前缀
–sslcertfile=FILE 可选客户端证书
–sslcertkey=KEYFILE 可选客户端证书的KEYFILE
–egd-file=FILE 指定EGD socket的文件名

----

下载参数：
–bind-address=ADDRESS 指定本地使用地址(主机名或IP，当本地有多个IP或名字时使用)
**-t, –tries=NUMBER 设定最大尝试链接次数(0 表示无限制).**
-O –output-document=FILE 把文档写到FILE文件中
-nc, –no-clobber 不要覆盖存在的文件或使用.#前缀
**-c, –continue 接着下载没下载完的文件**
–progress=TYPE 设定进程条标记
**-N, –timestamping 不要重新下载文件除非比本地文件新**
-S, –server-response 打印服务器的回应
–spider 不下载任何东西
-T, –timeout=SECONDS 设定响应超时的秒数
-w, –wait=SECONDS 两次尝试之间间隔SECONDS秒
–waitretry=SECONDS 在重新链接之间等待1…SECONDS秒
–random-wait 在下载之间等待0…2*WAIT秒
-Y, –proxy=on/off 打开或关闭代理
-Q, –quota=NUMBER 设置下载的容量限制
–limit-rate=RATE 限定下载输率

----

目录参数：
-nd –no-directories 不创建目录
**-x, –force-directories 强制创建目录**
-nH, –no-host-directories 不创建主机目录
**-P, –directory-prefix=PREFIX 将文件保存到目录 PREFIX/…**
–cut-dirs=NUMBER 忽略 NUMBER层远程目录

----

HTTP 选项参数：
–http-user=USER 设定HTTP用户名为 USER.
–http-passwd=PASS 设定http密码为 PASS
-C, –cache=on/off 允许/不允许服务器端的数据缓存 (一般情况下允许)
-E, –html-extension 将所有text/html文档以.html扩展名保存
–ignore-length 忽略 `Content-Length’头域
–header=STRING 在headers中插入字符串 STRING
–proxy-user=USER 设定代理的用户名为 USER
–proxy-passwd=PASS 设定代理的密码为 PASS
**–referer=URL 在HTTP请求中包含 `Referer: URL’头**
-s, –save-headers 保存HTTP头到文件
**-U, –user-agent=AGENT 设定代理的名称为 AGENT而不是 Wget/VERSION**
–no-http-keep-alive 关闭 HTTP活动链接 (永远链接)
–cookies=off 不使用 cookies
–load-cookies=FILE 在开始会话前从文件 FILE中加载cookie
–save-cookies=FILE 在会话结束后将 cookies保存到 FILE文件中

----

FTP 选项参数：
-nr, –dont-remove-listing 不移走 `.listing’文件
-g, –glob=on/off 打开或关闭文件名的 globbing机制
–passive-ftp 使用被动传输模式 (缺省值).
–active-ftp 使用主动传输模式
–retr-symlinks 在递归的时候，将链接指向文件(而不是目录)

----

**递归下载参数：**
**-r, –recursive 递归下载－－慎用!
-l, –level=NUMBER 最大递归深度 (inf 或 0 代表无穷)**
–delete-after 在现在完毕后局部删除文件
**-k, –convert-links 转换非相对链接为相对链接**
-K, –backup-converted 在转换文件X之前，将之备份为 X.orig
-m, –mirror 等价于 -r -N -l inf -nr
**-p, –page-requisites 下载显示HTML文件的所有图片**

递归下载中的包含和不包含(accept/reject)：
**-A, –accept=LIST 分号分隔的被接受扩展名的列表
-R, –reject=LIST 分号分隔的不被接受的扩展名的列表
-D, –domains=LIST 分号分隔的被接受域的列表
–exclude-domains=LIST 分号分隔的不被接受的域的列表**
–follow-ftp 跟踪HTML文档中的FTP链接
–follow-tags=LIST 分号分隔的被跟踪的HTML标签的列表
-G, –ignore-tags=LIST 分号分隔的被忽略的HTML标签的列表
-H, –span-hosts 当递归时转到外部主机
**-L, –relative 仅仅跟踪相对链接**
**-I, –include-directories=LIST 允许目录的列表
-X, –exclude-directories=LIST 不被包含目录的列表**
-np, –no-parent 不要追溯到父目录
wget -S –spider url 不下载只显示过程


----
###  使用实例： 
使用wget下载单个文件
wget http://www.minjieren.com/wordpress-3.1-zh_CN.zip

----

使用wget -O下载并以不同的文件名保存
wget -O wordpress.zip http://www.minjieren.com/download.aspx?id=1080
说明：
` wget默认会以最后一个符合”/”的后面的字符来命令，对于动态链接的下载通常文件名会不正确。 `
错误：下面的例子会下载一个文件并以名称download.aspx?id=1080保存
wget http://www.minjieren.com/download?id=1
即使下载的文件是zip格式，它仍然以download.php?id=1080命令。
正确：为了解决这个问题，我们可以使用参数-O来指定一个文件名：
wget -O wordpress.zip http://www.minjieren.com/download.aspx?id=1080

----

使用wget –limit-rate限速下载
```

wget --limit-rate=300k http://www.minjieren.com/wordpress-3.1-zh_CN.zip

```

----

使用wget -c断点续传
wget -c http://www.minjieren.com/wordpress-3.1-zh_CN.zip
使用wget -c重新启动下载中断的文件，对于我们下载大文件时突然由于网络等原因中断非常有帮助，我们可以继续接着下载而不是重新下载一个文件。需要继续中断的下载时可以使用-c参数。

----

使用wget -b后台下载
wget -b http://www.minjieren.com/wordpress-3.1-zh_CN.zip
说明：
对于下载非常大的文件的时候，我们可以使用参数-b进行后台下载。
wget -b http://www.minjieren.com/wordpress-3.1-zh_CN.zip
Continuing in background, pid 1840.
Output will be written to `wget-log'.
你可以使用以下命令来察看下载进度：
tail -f wget-log

----
伪装代理名称下载
```

wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.204 Safari/534.16" http://www.minjieren.com/wordpress-3.1-zh_CN.zip

```

----
使用wget –spider测试下载链接
wget --spider URL
说明：
你可以在以下几种情况下使用spider参数：
  * 定时下载之前进行检查
  * 间隔检测网站是否可用
  * 检查网站页面的死链接

----
使用wget -i下载多个文件
wget -i filelist.txt
说明：
首先，保存一份下载链接文件
cat > filelist.txt
url1
url2
url3
url4
接着使用这个文件和参数-i下载

----
使用wget –mirror镜像网站
```

wget --mirror -p --convert-links -P ./LOCAL URL

```
说明：
下载整个网站到本地。
–miror:开户镜像下载
-p:下载所有为了html页面显示正常的文件
–convert-links:下载后，转换成本地的链接
-P ./LOCAL：保存所有文件和目录到本地指定目录

----

使用wget –reject过滤指定格式下载
```

wget --reject=gif url

```
说明：
下载一个网站，但你不希望下载图片，可以使用以下命令。

----
使用wget -r -A下载指定格式文件
```

wget -r -A.pdf url

```
说明：
可以在以下情况使用该功能：
  * 下载一个网站的所有图片
  * 下载一个网站的所有视频
  * 下载一个网站的所有PDF文件

----

使用wget FTP下载
命令：
```

wget ftp-url
wget --ftp-user=USERNAME --ftp-password=PASSWORD url

```
说明：
可以使用wget来完成ftp链接的下载。
使用wget匿名ftp下载：
wget ftp-url


----
###  备注：编译安装 
使用如下命令编译安装： 
```

# tar zxvf wget-1.9.1.tar.gz 
# cd wget-1.9.1 
# ./configure 
# make 
# make install

```
##  curl 
###  使用curl指令測試REST服務 
cURL 是很方便的Rest客戶端，可以很方便的完成許多Rest API測試的需求.
curl的參數很多，這邊僅列出目前測試REST時常用到的:
  * -X/--request [GET|POST|PUT|DELETE|…]  使用指定的http method發出 http request
  * -H/--header                           設定request裡的header
  * -d/--data                             設定 http parameters 
  * -v/--verbose                          輸出比較多的訊息
  * -u/--user                             使用者帳號、密碼
  * -b/--cookie                           cookie  
  * -I  header信息
  * -E cert.pem  指定本地证书
----

GET/POST/PUT/DELETE使用方式
-X 後面加 http method，
```

curl -X GET "http://www.rest.com/api/users"
curl -X POST "http://www.rest.com/api/users"
curl -X PUT "http://www.rest.com/api/users"
curl -X DELETE "http://www.rest.com/api/users"

```

----

HEADER
在http header加入的訊息
```

curl -v -i -H "Content-Type: application/json" http://www.example.com/users

```

----

HTTP Parameter
http參數可以直接加在url的query string，也可以用-d帶入參數間用&串接，或使用多個-d
```

# 使用`&`串接多個參數
curl -X POST -d "param1=value1&param2=value2"
# 也可使用多個`-d`，效果同上
curl -X POST -d "param1=value1" -d "param2=value2"
curl -X POST -d "param1=a 0space"     
# "a space" url encode後空白字元會編碼成'%20'為"a%20space"，編碼後的參數可以直接使用
curl -X POST -d "param1=a%20space"

```

----

post json 格式得資料
如同時需要傳送request parameter跟json，request parameter可以加在url後面，json資料則放入-d的參數，然後利用單引號將json資料含起來(如果json內容是用單引號，-d的參數則改用雙引號包覆)，header要加入”Content-Type:application/json”跟”Accept:application/json”
```

curl http://www.example.com?modifier=kent -X PUT -i -H "Content-Type:application/json" -H "Accept:application/json" -d '{"boolean" : false, "foo" : "bar"}'
# 不加"Accept:application/json"也可以
curl http://www.example.com?modifier=kent -X PUT -i -H "Content-Type:application/json" -d '{"boolean" : false, "foo" : "bar"}'

```

----

需先認證或登入才能使用的service
許多服務，需先進行登入或認證後，才能存取其API服務，依服務要求的條件，的curl可以透過cookie，session或加入在header加入session key，api key或認證的token來達到認證的效果。
session 例子:
後端如果是用session記錄使用者登入資訊，後端會傳一個 session id給前端，前端需要在每次跟後端的requests的header中置入此session id，後端便會以此session id識別前端是屬於那個session，以達到session的效果
```

curl --request GET 'http://www.rest.com/api/users' --header 'sessionid:1234567890987654321'

```

----
cookie 例子
如果是使用cookie，在認證後，後端會回一個cookie回來，把該cookie成檔案，當要存取需要任務的url時，再用-b cookie_file 的方式在request中植入cookie即可正常使用
```

# 將cookie存檔
curl -i -X POST -d username=kent -d password=kent123 -c  ~/cookie.txt  http://www.rest.com/auth
# 載入cookie到request中 
curl -i --header "Accept:application/json" -X GET -b ~/cookie.txt http://www.rest.com/users/1

```

----
檔案上傳
```

curl -i -X POST -F 'file=@/Users/kent/my_file.txt' -F 'name=a_file_name'

```
這個是透過 HTTP multipart POST 上傳資料， -F 是使用http query parameter的方式，指定檔案位置的參數要加上@

----
HTTP Basic Authentication (HTTP基本認證)
如果網站是採HTTP基本認證, 可以使用 --user username:password 登入
```

curl -i --user kent:secret http://www.rest.com/api/foo'

```

----
###  curl抓取网页 
抓取百度：
curl http://www.baidu.com 
如发现乱码，可以使用iconv转码：
curl http://iframe.ip138.com/ic.asp|iconv -f gb2312

----

Linux curl使用代理：
linux curl使用http代理抓取页面：

curl -x 111.95.243.36:80 http://iframe.ip138.com/ic.asp|iconv -fgb2312
curl -x 111.95.243.36:80 -U aiezu:password http://www.baidu.com
使用socks代理抓取页面：
curl --socks4 202.113.65.229:443 http://iframe.ip138.com/ic.asp|iconv -fgb2312
curl --socks5 202.113.65.229:443 http://iframe.ip138.com/ic.asp|iconv -fgb2312

##  使用curl下载文件 
curl -o [filename] <url>



参考：http://ju.outofmemory.cn/entry/84875
http://www.aiezu.com/system/linux/linux_curl_syntax.html
http://www.linuxidc.com/Linux/2014-09/107018.htm