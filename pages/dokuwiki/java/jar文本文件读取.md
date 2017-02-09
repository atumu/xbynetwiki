title: jar文本文件读取 

#  jar文本文件读取 

**使用工程相对路径是靠不住的。
使用CLASSPATH路径是可靠的。**
对于程序要读取的文件，尽可能放到CLASSPATH下，这样就能保证在开发和
发布时候均正常读取。
InputStream in = ReadFile.class.getResourceAsStream("/com/lavasoft/res/a.txt");
有了字节流，就能读取到文件内容了。
注意：
**这里必须以“/”开头；**