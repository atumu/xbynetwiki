title: js日期格式化 

#  js日期格式化 
可供参考：http://my.oschina.net/xiang1987/blog/155221
但是本文采用一种便捷方式：
```

//服务端返回｛"articlePublishTime"1441641600000｝
var datePreStr=e.articlePublishTime;
var dateObj=new Date(datePreStr);
var createTime=dateObj.getFullYear()+"年"+dateObj.getMonth()+"月"+dateObj.getDay()+"日"+dateObj.getHours()+":"+dateObj.getMinutes()+":"+dateObj.getSeconds();

```