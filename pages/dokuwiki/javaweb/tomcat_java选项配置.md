title: tomcat_java选项配置 

#  tomcat java选项配置 
修改catalina.bat添加:
set JAVA_OPTS= -server -XX:PermSize=256M -XX:MaxPermSize=512m -XX:-UseSplitVerifier