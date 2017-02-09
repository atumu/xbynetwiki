title: springmvc使用问题集锦 

#  SpringMVC使用问题集锦 
1、出现POST 404 Bad Request：
原因是我Controller中接收数据的对象中含有Timestamp类型属性。而ajax一般只是用于传输字符串类型值，所以对于SpringMVC就无法识别和转换了。
解决办法，将接受前台数据的Dto对象中Timestamp改为String即可。
参考：http://www.myexception.cn/operating-system/1867084.html
http://aokunsang.iteye.com/blog/1878985