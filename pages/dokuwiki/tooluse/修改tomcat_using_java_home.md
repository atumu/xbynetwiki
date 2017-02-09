title: 修改tomcat_using_java_home 

#  Linux下修改tomcat using java_home 
参考：http://www.educity.cn/wenda/211390.html
llinux 运行tomcat时JRE_HOME显示不对 怎么办？
Using CATALINA_BASE:   /usr/share/tomcat7
Using CATALINA_HOME:   /usr/share/tomcat7
Using CATALINA_TMPDIR: /usr/share/tomcat7/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /usr/share/tomcat7/bin/bootstrap.jar:/usr/share/tomcat7/bin/tomcat-juli.jar
pyt@pyt-Ideapad-S205:/usr/share/tomcat7/bin$ 

看下Tomcat的` startup.sh `，启动的时候它调用了catalina.sh,而` catalina.sh `则调用了` setclasspath.sh `。
只要在` setclasspath.sh `声明环境变量就可以知道你这个tomcat使用哪个jdk，
打开tomcat的bin目录下面的setclasspath.sh，添加上，路径自己修改，**添加在开头就行**
export JAVA_HOME=/usr/lib/jvm/jdk7
export JRE_HOME=/usr/lib/jvm/jdk7