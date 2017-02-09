title: hibernate_jpa配置文件放置路径 

#  JPA配置文件放置路径 
最近做一个web项目用到了Spring+JPA，由于没有正确配置persistence.xml的文件路径，导致出现如下错误：
No persistence units parsed from {classpath*:META-INF/persistence.xml}
查找原因，原来在web工程文件夹下，本来有一个 META-INF 文件夹，但这个文件夹是和 WEB-INF 目录同级：
![](/data/dokuwiki/hibernate/pasted/20150908-174438.png)
JPA的persistence.xml配置文件不能放到这个META-INF文件夹下，而是要放到`  WEB-INF/classes/META-INF `文件夹下：
![](/data/dokuwiki/hibernate/pasted/20150908-174450.png)
这样再启动tomcat，访问web工程就不会出现上述错误了。

还有一个要注意的是` persistencen.xml放在classpath的不同地方也决定了持久化单元的作用域。 `
