title: nginx问题配置1 

#  解决采用nginx反向代理tomcat时JSP动态页面不刷新问题 
采用默认配置的nginx反向代理tomcat，发现当页面操作完成并跳转之后，跳转页面的内容不会实时刷新，也就是我们看到的仍然是操作完成之前的“旧页面”的内容。
经过测试发现，直接使用tomcat不会有这个问题，也就是说nginx对tomcat进行了缓存，没有实时刷新页面。因此，我们需要对nginx.conf进行配置，解决缓存问题，具体方法如下：
```

<!-- lang: shell -->
    server_name localhost:8080;
    location ~ \.jsp$ {
            proxy_buffering off;;
            proxy_pass http://localhost:8080;
    }

```

加上 proxy_buffering off; 就可以解决问题。

http://www.codeweblog.com/%E8%A7%A3%E5%86%B3%E9%87%87%E7%94%A8nginx%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86tomcat%E6%97%B6jsp%E5%8A%A8%E6%80%81%E9%A1%B5%E9%9D%A2%E4%B8%8D%E5%88%B7%E6%96%B0%E9%97%AE%E9%A2%98/