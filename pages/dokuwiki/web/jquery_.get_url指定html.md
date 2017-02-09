title: jquery_.get_url指定html 

#  jQuery $.get() url指定html 
y也可以称之为jQuery怎么加载一个html页面到我指定的div里面
参考：http://blog.csdn.net/chelen_jak/article/details/34434431
一、juqery $.ajax 请求另一个html页面的指定的“一部分”加载到本页面div，重点是一部分数据加载到本页面div  (来自百度知道)
大至思路如下：
```

$.ajax( {
        url: 'latestNewsInput.html', //这里是静态页的地址
        type: "GET", //静态页用get方法，否则服务器会抛出405错误
        success: function(data){
            var result = $(data).find("另一个html页面的指定的一部分");
            $("本页面div").html(result);


        }
});

```

另一种方式：
```

$.get(getPublishServer()+ '/plugin/latestNews/latestNewsInput.html', function(newsTemp) {});

```