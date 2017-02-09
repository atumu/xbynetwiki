title: 解决docker容器内时间不同步 

#  解决docker容器内JVM时间不一致问题 
##  解决JVM时区问题 
 方式一、（不推荐，date正常，但是在产生日志文件时日志文件的时间还是存在问题）
docker run ` -v /etc/localtime:/etc/localtime:ro `
接用宿主主机的时间

方式二、推荐
修改时区[[linux:ubuntu修改时区]]

方式三、不推荐，每启用一个JVM都要进行配置：
造成这种问题的原因可能是：**你的操作系统时区跟你JVM的时区不一致。** 
你的操作系统应该是中国的时区吧，而JVM的时区不一定是中国时区，
你在应用服务器的Java虚拟机添加如下配置：  ` -Duser.timezone=GMT+08 `

参考：http://phosphory.iteye.com/blog/518868
http://blog.sina.com.cn/s/blog_684fe8af0100ww36.html

##  解决tomcat时区问题 
在catalina.sh 第一行加入
JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF8  -Duser.timezone=GMT+08"