author:xbynet
title:mdwiki开发之路一技术选型
modifyAt:2016-12-12 03:20:44
location:proj/mdwiki开发之路一技术选型
createAt:2016-12-12 03:20:44

mdwiki是一款markdown wiki系统，可以作为个人或小型团队的知识库管理系统。项目地址：本系列文章最后一篇给出(需要时间整理和测试)

为什么我要开发mdwiki?
--------------

目前本人的知识库管理系统采用的是dokuwiki，它是一款用PHP开发的非常强大的一款wiki系统。但是很遗憾不支持markdown语法写作。
再加上目前开始学习Python与爬虫。所以决定用Python写一个markdown wiki系统。前期不考虑集成爬虫，后期考虑集成爬虫(这样对某些好文章的收藏就没必要复制粘贴了)。

技术选型
----
Python3 or 2.7？

作为新手，Python3义不容辞.为什么？就为了原生支持UTF-8.(开玩笑),因为Python3代表了Python的未来，而且越来越多的库已经迁移到了Python3，没有什么理由不选择它。

IDE选择：

pycharm+sublime text3，这个也没必要解释了。

Web框架选择:

Flask(为什么？只会这个，而且大家都说好。)

服务器选择:

nginx+gunicorn这应该是比较流行的方案吧，也不做过多解释。

数据库选择：
SQLite+Redis

部署方式:
Supervisor管理Nginx+gunicorn
Fabric远程发布

浏览器兼容性：
不考虑万恶的IE

后端库选择:
Flask Web框架
Jinja2 flask官方指定模板引擎
SQLAlchemy ORM框架
Celery任务调度
whoosh+jieba:信息检索
oss2：阿里云oss云存储SDK
redis：Redis的python连接客户端
Markdown：后端markdown解析

Flask插件如下：
Flask-Babel国际化插件
Flask-Script命令行插件
Flask-sqlalchemy ORM插件集成
Flask-migrate数据迁移插件
Flask-WTF表单插件
flask-login插件
flask-Principal权限管理
Flask-Security插件
flask-mail插件
Flask-cache缓存插件
flask-testing测试插件
Flask-Moment本地化时间日期


前端库选择：

gulp-前端资源管理与打包,可以参考我的一篇文章:[gulp组织小型项目小记][1]

animate.css特效
bootstrap 都懂的
jQuery
jQuery插件：validate,fancyBox,jQuery-ui
simplemde markdown编辑器
webuploader 百度开源的文件上传组件
toastr一款漂亮的通知组件
taggle.js 标签输入组件
highlight.js代码高亮

下一篇主题待定.


  [1]: https://segmentfault.com/a/1190000007613571