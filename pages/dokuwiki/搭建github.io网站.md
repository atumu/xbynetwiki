title: 搭建github.io网站 

#  搭建github.io网站 
官方教程：https://pages.github.com/
Github很好的将代码和社区联系在了一起，于是发生了很多有趣的事情，世界也因为他美好了一点点。Github作为现在最流行的代码仓库，已经得到很多大公司和项目的青睐，比如jQuery、Twitter等。为使项目更方便的被人理解，介绍页面少不了，甚至会需要完整的文档站，Github替你想到了这一点，他提供了**Github Pages**的服务，不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。
Github Pages有以下几个优点：

  * 轻量级的博客系统，没有麻烦的配置
  * 使用标记语言，比如Markdown
  * 无需自己搭建服务器
  * 根据Github的限制，对应的每个站有300MB空间
  * 可以绑定自己的域名

当然他也有缺点：

  * 使用Jekyll模板系统，相当于静态页发布，适合博客，文档介绍等。
  * 动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。
  * 基于Git，很多东西需要动手，不像Wordpress有强大的后台

##  使用GitHub Pages建立博客 
与GitHub建立好链接之后，就可以方便的使用它提供的Pages服务，GitHub Pages分两种，一种是你的GitHub用户名建立的username.github.io这样的用户&组织页（站），另一种是依附项目的pages。
User & Organization Pages
想建立个人博客是用的第一种，形如beiyuu.github.io这样的可访问的站，每个用户名下面只能建立一个，创建之后点击Settings进入项目管理，可以看到是这样的：
![](/data/dokuwiki/pasted/20150531-090728.png)
点击下面这个按钮初始化：
![](/data/dokuwiki/pasted/20150531-090755.png)
即可通过youname.githun.io进入你的站点

关于模版系统说明请参考相关文档
参考：
http://beiyuu.com/github-pages/