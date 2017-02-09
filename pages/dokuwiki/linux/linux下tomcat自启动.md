title: linux下tomcat自启动 

#  linux下tomcat自启动 
如果是ubuntu，先配置默认JDK:
配置默认JDK
由于一些Linux的发行版中已经存在默认的JDK，如OpenJDK等。所以为了使得我们刚才安装好的JDK版本能成为默认的JDK版本，我们还要进行下面的配置。
执行下面的命令：
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk7/bin/java 300 
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk7/bin/javac 300

**linux下无法启动tomcat** 请参考[Linux下修改tomcat using java_home](/pages/dokuwiki/tooluse/修改tomcat using java_home)

安装tomcat 使用命令：tar zvxf apache-tomcat-6.0.28.tar.gz
切换路径到tomcat的bin目录下。
**Tomcat自启动：**
```

cp catalina.sh /etc/init.d/tomcat
sudo chmod 755 /etc/init.d/tomcat (可选，默认就是755)

```
打开/etc/init.d/tomcat
加入：
  * CATALINA_HOME=/zjc/tomcat(tomcat主目录)
  * JAVA_HOME=/zjc/jdk1.6.0_27 （jdk主目录）  
保存后，可以执行**service tomcat start ** 
```

CATALINA_HOME=/opt/tomcat7
JAVA_HOME=/opt/jdk7
###在这前面加入 CATALINA_HOME JAVA_HOME
# OS specific support.  $var _must_ be set to either true or false.
cygwin=false
darwin=false
os400=false
case "`uname`" in
CYGWIN*) cygwin=true;;
Darwin*) darwin=true;;
OS400*) os400=true;;
esac

```
成功后，**添加到系统自启动服务**：
```

update-rc.d  -f  tomcat  defaults

```
删除自启动：
```

udpate-rc.d –f tomcat remove.

```

参考：http://jingyan.baidu.com/article/8cdccae964de5d315413cd27.html