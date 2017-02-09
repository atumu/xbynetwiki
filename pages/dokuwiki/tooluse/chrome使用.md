title: chrome使用 

#  chrome使用 
1、Chrome调试网页如何禁用缓存？
http://jingyan.baidu.com/article/574c5219cbedec6c8d9dc1bd.html
这几天在调试我的一个虚拟主机，因为带宽有限所以，有些文件我需要查看他的加载速度然后进行优化。那么如何在用Chrome调试的时候禁用缓存呢？

下面是浏览器使用到了缓存的时候 显示304 （不利于查看真正的文件加载时间）
![](/data/dokuwiki/tooluse/pasted/20150902-113501.png)
打开 Chrome调试工具。按F12键。

然后 点击右上角的设置。
![](/data/dokuwiki/tooluse/pasted/20150902-113512.png)
再打开的网页中

选择Disable cache (while DevTools is open)

这样的话。
![](/data/dokuwiki/tooluse/pasted/20150902-113525.png)
在重新加载的时候。会提示200.
![](/data/dokuwiki/tooluse/pasted/20150902-113542.png)