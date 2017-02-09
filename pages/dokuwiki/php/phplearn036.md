title: phplearn036 

#  PHP学习之Xdebug与Sublime+Chrome调试 
1、安装xdebug扩展，配置php.ini:
```

[XDebug]
zend_extension = "C:\Users\taojw\Desktop\learn\nginx-1.9.7\php\ext\php_xdebug.dll"
xdebug.remote_enable=1
;xdebug.remote_host="127.0.0.1"
xdebug.remote_port=9001 ;不要设置为默认的9000，这样会与fastcgi进程端口冲突
xdebug.remote_handler="dbgp"
xdebug.remote_mode = req
xdebug.remote_connect_back = 1

```
2、安装sublime xdebug插件
3、安装chrome浏览器插件（主要用途就是不用输入url调试参数http://localhost:4001/task?XDEBUG_SESSION_START=sublime.xdebug,而是直接输入http://localhost:4001/task）
安装chrome插件.Xdebug helper 配置xdebug helper .打开chrome扩展选项.
![](/data/dokuwiki/php/pasted/20160427-152021.png)
保存，重启下chrome .  firefox也有插件.

4、用sublime打开你要调试的程序，点击sublime导航的` Project->save project as `。生成一个.sublime-project的文件，然后点击` project->edit poject `修改其为：
如果是本机调试，配置文件类似以下内容：
```

{
    "folders":
    [
        {
           // "follow_symlinks": true,
            "path": "."
        }
    ],
    "settings": {
        "xdebug": {
          //"path_mapping": {   #本机调试此项不需要设置  
           // },  
             "url": "http://localhost:4001/",
             // "super_globals": true,  
            //"close_on_stop": true, 
             "port": 9001   #此port与之前xdebug扩展一致  
        }
    }
}

```
如果是远程调试:
```

{  
    "folders":  
    [  
        {  
            "path": "/D/biwebs"  
        }  
    ],  
    "settings":  
    {  
        "xdebug": {  
            "path_mapping": {  
                "/absolute/path/to/file/on/server" : "/absolute/path/to/file/on/computer",  #与本地就此处不同，必须将远程与本地的映射写明  
            },  
            "url": "http://wiki.123.net/test.php",  
            "super_globals": true,  
            "close_on_stop": true,  
            "port": 9001  
        }  
    }  
}  

```

5、设置断点调试：
先在试例代码中标记个断点(ctrl+F8)
![](/data/dokuwiki/php/pasted/20160427-153814.png)
打开浏览器输入 http://localhost:4001/?XDEBUG_SESSION_START=sublime.xdebug
![](/data/dokuwiki/php/pasted/20160427-153843.png)
![](/data/dokuwiki/php/pasted/20160427-153924.png)

**如果是Laravel的调试**，请进一步参考:http://blog.osteel.me/posts/2015/05/23/laravel-homestead-debug-an-api-with-xdebug-and-curl-in-sublime-text.html

![](/data/dokuwiki/php/pasted/20160427-154531.png)
sublimexdebug的一些快捷键
  * Shift+f8: 打开调试面板
  * f8:打开调试面板快速连接
  * Ctrl+f8: 切换断点
  * Ctrl+Shift+f5: 运行到下一个断点
  * Ctrl+Shift+f6: 单步
  * Ctrl+Shift+f7: 步入
  * Ctrl+Shift+f8: 步出 
参考：http://mattwatson.codes/debug-php-vagrant-using-xdebug-sublime-text/
http://yansu.org/2014/03/20/php-debug-with-xdebug.html
https://github.com/martomo/SublimeTextXdebug
http://blog.csdn.net/liuzhushiqiang/article/details/14170483
http://lobert.iteye.com/blog/2068638