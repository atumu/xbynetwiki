title: 判断一个请求是否为ajax请求 

#  判断一个请求是否为AJAX请求 
普通请求与ajax请求的报文头不一样，通过如下 
String requestType = request.getHeader("X-Requested-With");  
如果requestType能拿到值，并且值为XMLHttpRequest,表示客户端的请求为异步请求，那自然是ajax请求了，反之如果为null,则是普通的请求 