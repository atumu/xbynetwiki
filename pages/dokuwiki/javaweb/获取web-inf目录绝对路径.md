title: 获取web-inf目录绝对路径 

#  获取WEB-INF目录绝对路径 
String web_inf = getServletContext().getRealPath("/WEB-INF/"); 获取到WEB-INF 的地址，剩下的路径自己拼接吧。
如果是 classes下面的配置文件，可以直接使用
Thread.currentThread().getContextClassLoader().getResource("").getPath();  剩下的路径，就自己拼接吧

在windows下需要注意的是：如：/D:/ideawork/ws1/out/production/R3-Query-src/WEB-INF/classes
```

 String path=Thread.currentThread().getContextClassLoader().getResource("").getPath();
 path = path.substring(1, path.indexOf("classes"));
path=path.replace('/',File.separatorChar); // 将/换成\

```

参考：http://blog.csdn.net/magi1201/article/details/18731581
http://blog.csdn.net/friendan/article/details/19839767