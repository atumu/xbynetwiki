title: 使用fragment参数 

#  Chapter10 使用Fragment参数 
Created Wednesday 18 March 2015

##  从Fragment传递参数： 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085147.png)

##  获取extra信息 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085151.png)
但是这种直接获取extra信息的方式有很大的不足。那就是牺牲了Fragment的封装性，使之不再是可复用的构建单元。

#  fragment argument 
常用的附加argument给fragment的方式：
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085203.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085209.png)

##  获取argument 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085214.png)
#  重新加载显示列表项 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085219.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085223.png)

#  通过Fragment获取返回结果 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085229.png)
但是需要注意的是Fragment可以接受返回结果，但是Fragment不能设置返回结果。只有Activity拥有返回结果。故而需要进行如下方式设置返回结果。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085235.png)
