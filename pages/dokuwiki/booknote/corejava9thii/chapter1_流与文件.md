title: chapter1_流与文件 

#  Chapter1 流与文件 
Created Thursday 19 March 2015

#  主要知识点 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082427.png)

##  注意几点 

###  关于流中写出write于读入read 
读入是指从流中的数据读入到方法参数中的数据结构中。
写出是指将方法参数中的数据写出到流中。

###  关于java.io中的路径问题 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082433.png)

###  读入写出普通文件 
读入用Scanner或者BufferedReader
写出用PrintWriter或者DataOutputStream

#  流 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082441.png)
读写字节：
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082455.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082513.png)
##  组合流过滤器或者成为包装器（修饰） 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082529.png)

#  文本输入与输出 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082545.png)

##  如何写出文本输出 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082600.png)

##  如何读入文本输入 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082619.png)

##  字符集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082638.png)

#  读写二进制数据 

##  随机访问文件 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082654.png)

#  zip文档 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082711.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082718.png)
#  对象流与序列化 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082736.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082750.png)
##  修改默认的序列号机制与transient 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082808.png)

此时我们就应该用自己的writeObject或者readObject来进行序列化
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082827.png)

##  序列化单例和类型安全的枚举（重点） 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082855.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082907.png)
##  版本管理 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082919.png)

##  为克隆使用序列化 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082925.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082930.png)

#  操作文件 
java7开始引入的
java.nio.file.Paths,
java.nio.file.Path,
java.nio.file.Files
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082945.png)

##  读写文件 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-082958.png)

##  复制、移动和删除文件 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083004.png)

##  创建文件和目录 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083020.png)

##  获取文件信息 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083035.png)

##  迭代目录中的文件 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083058.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083122.png)
##  ZIP文件系统 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083128.png)
使用FileSystems创建文件系统FileSystem
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083133.png)

```

public class ZipFS {
	public static final String path =System.getProperty("user.home")+"/Desktop/io.jar";
	public static void main(String[] args) throws Exception{
		System.out.println(path);
		FileSystem fs=FileSystems.newFileSystem(Paths.get(path), null);
		Files.walkFileTree(fs.getPath("/"), new SimpleFileVisitor<Path>(){
			public FileVisitResult visitFile(Path file,BasicFileAttributes attrs) throws IOException{
				System.out.println(file);
				return FileVisitResult.CONTINUE;
			}
		});
	}
}

```
#  内存映射文件 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083219.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083242.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083304.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083319.png)
##  缓冲区数据结构 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083425.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083356.png)
要想获取缓冲区，可以调用诸如ByteBuffer.allocate或ByteBuffer.wrap。

##  文件加锁机制 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083503.png)

#  正则表达式 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083526.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083543.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083600.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083616.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083633.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-083647.png)