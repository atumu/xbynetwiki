author:xbynet
title:mdwiki开发之路二资源与踩坑记录
modifyAt:2016-12-12 03:21:45
location:proj/mdwiki开发之路二资源与踩坑记录
createAt:2016-12-12 03:21:45

1、[bootstrap代码片段][1]：
---------------------
如果你没有艺术细胞，偷懒的方法就是到这上面去找，比如登录框界面等。
侧边栏选用：http://www.designerslib.com/bootstrap-sidebar-menu-templates/提到的http://bootsnipp.com/fullscreen/4OjXB。
其他一些资源：
[w3schools-howto][2]
[一个比较炫的html模板(虽然最后没有采用)][3]
[bootstrap主题][4]

2、DIV的CSS height:100%无效的解决办法：
-----------------------------

在css当中增加上：

    html, body{ margin:0; height:100%; }


3、[Alembic migration失败，Sqlite lack of ALTER support解决办法：][5]
------------------------------------------------------------

在env.py中设置`render_as_batch=True`

    context.configure(
        connection=connection,
        target_metadata=target_metadata,
        render_as_batch=True
    )

4、[markdown扩展][6]:
------------------

http://pythonhosted.org/Markdown/extensions/
比较有用的
Table of Contents(toc)、
CodeHilite(代码高亮)、
Meta-Data(文件前面可以添加元数据，比如标题，作者等)、
New Line to Break(换行即新行，而不是像原生markdown那样得换两行)、
Tables(表格插件)

5、关于Flask的：
-----------

[Flask request，g，session的实现原理][7]
[深入 Flask 源码理解 Context][8]
[Flask Session超时设置][9]
默认情况下，flask session在你关闭浏览器的时候失效。你可以通过设置permanent session来改变这一行为。

    from datetime import timedelta
    from flask import session, app
    
    @app.before_request
    def make_session_permanent():
        session.permanent = True
        app.permanent_session_lifetime = timedelta(minutes=30)

默认情况下，permanent_session_lifetime是31天。

6、关于SQLAlchemy:
---------------

[SQLAlchemy 使用经验][10]
[SqlAlchemy query many-to-many relationship][11]

    class Restaurant(db.Model):
        ...
    
        dishes = db.relationship('Dish', secondary=restaurant_dish,
            backref=db.backref('restaurants'))

然后检索所有的dishes for a restaurant, you can do:

    x = Dish.query.filter(Dish.restaurants.any(name=name)).all()

产生类似如下SQL语句：

    SELECT dish.*
    FROM dish
    WHERE
        EXISTS (
            SELECT 1
            FROM restaurant_dish
            WHERE
                dish.id = restaurant_dish.dish_id
                AND EXISTS (
                    SELECT 1
                    FROM restaurant
                    WHERE
                        restaurant_dish.restaurant_id = restaurant.id
                        AND restaurant.name = :name
                )
        )
    
    

7、[解决循环import的问题思路][12]
-----------------------

1.延迟导入(lazy import)
即把import语句写在方法或函数里面，将它的作用域限制在局部。
这种方法的缺点就是会有性能问题。
2.将from xxx import yyy改成import xxx;xxx.yyy来访问的形式
3.组织代码
出现循环import的问题往往意味着代码的布局有问题，可以合并或者分离竞争资源。合并的话就是都写到一个文件里面去。分离的话就是把需要import的资源提取到一个第三方文件去。总之就是 将循环变成单向。
具体解决方案后续文章再贴代码


8、关于Python的一些：
--------------

[Good logging practice in Python][13]
[How do I check if a variable exists?][14]
To check the existence of a local variable:

    if 'myVar' in locals():
      # myVar exists.

To check the existence of a global variable:

    if 'myVar' in globals():
      # myVar exists.

To check if an object has an attribute:

    if hasattr(obj, 'attr_name'):
      # obj.attr_name exists.
    if('attr_name' in dir(obj)):
        pass
      
还有一个不是很优雅地方案,通过捕获异常的方式：

```
try:
    myVar
except NameError:
    myVar = None
# Now you're free to use myVar without Python complaining.
```

9、关于Git与Github
--------------

[How do I delete a Git branch with TortoiseGit][15]
[为什么给GIT库打TAG不成功][16]

[如何修改github上仓库的项目语言?][17]

项目放在github，是不是经常被识别为javascript项目？知乎这篇问答给出了答案。
问题原因: 
github 是根据项目里文件数目最多的文件类型,识别项目类型.
解决办法:
项目根目录添加 `.gitattributes` 文件, 内容如下 :

    *.js linguist-language=python

作用: 把项目里的 .js 文件, 识别成 python 语言.

10、关于IDE的：
----------

[Indexing excluded directories in PyCharm][18]
[pycharm convert tabs to spaces automatically][19]

11、关于Celery的：
-------------

[periodic task for celery sent but not executed][20]
这个由于我没仔细看官方文档，搞了好久。Celery的周期性任务scheduler需要配置beat和运行beat进程,但是仅仅运行beat进程可以吗？不行！我就是这里被坑了。还得同时运行一个worker。也就是说beat和worker都需要通过命令行运行。对于周期性任务beat缺一不可。其他任务可仅运行worker。

12、[在supervisor或gunicorn设置环境变量][21]
-----------------------------------

如果采用gunicorn命令行的形式：-e选项

    gunicorn -w 4 -b 127.0.0.1:4000 -k gevent -e aliyun_api_key=value,SECRET_KEY=mysecretkey app:app 

如果采用gunicorn.conf.py文件的形式：raw_env

```
import multiprocessing

bind = "127.0.0.1:4000"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class='gevent'
proc_name = "mdwiki"
user = "nginx"
chdir='/opt/mdwiki'
#daemon=False
#group = "nginx"
loglevel = "info"
#errorlog = "/home/myproject/log/gunicorn.log"
#accesslog=
raw_env = [
   'aliyun_api_key=value',
   'aliyun_secret_key=value',
   'MAIL_PASSWORD=value',
   'SECRET_KEY=mysecretkey',
]
#ssl
#keyfile=
#certfile=
#ca_certs=
```

   
如果采用supervisor配置环境变量

```
[program:mdwiki]
environment=SECRET_KEY=value,aliyun_api_key=value,aliyun_secret_key=value,MAIL_PASSWORD=value
command=/usr/bin/gunicorn -n mdwiki -w 4 -b 127.0.0.1:4000 -k gevent app:app 
directory=/opt/mdwiki
user=nginx
autostart=true
autorestart=true
redirect_stderr=true
```


  [1]: http://bootsnipp.com/
  [2]: http://www.w3schools.com/howto/
  [3]: http://demo.cssmoban.com/cssthemes3/mbts_13_candidate/main-v1.html
  [4]: https://bootswatch.com/
  [5]: http://stackoverflow.com/questions/30378233/sqlite-lack-of-alter-support-alembic-migration-failing-because-of-this-solutio
  [6]: http://pythonhosted.org/Markdown/extensions/
  [7]: http://blog.csdn.net/yueguanghaidao/article/details/39533841
  [8]: http://mousehouse.applinzi.com/article/7
  [9]: http://stackoverflow.com/questions/11783025/is-there-an-easy-way-to-make-sessions-timeout-in-flask
  [10]: https://www.keakon.net/2012/12/03/SQLAlchemy%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C
  [11]: http://stackoverflow.com/questions/12593421/sqlalchemy-and-flask-how-to-query-many-to-many-relationship
  [12]: http://www.tuicool.com/articles/Bv2aEf
  [13]: https://fangpenlin.com/posts/2012/08/26/good-logging-practice-in-python/
  [14]: http://stackoverflow.com/questions/843277/how-do-i-check-if-a-variable-exists
  [15]: http://stackoverflow.com/questions/9705534/how-do-i-delete-a-git-branch-with-tortoisegit
  [16]: http://jingyan.baidu.com/article/5d6edee2f611fa99eadeec01.html
  [17]: https://www.zhihu.com/question/47249333
  [18]: http://stackoverflow.com/questions/19024553/indexing-excluded-directories-in-pycharm-3
  [19]: http://stackoverflow.com/questions/11816147/pycharm-convert-tabs-to-spaces-automatically
  [20]: http://stackoverflow.com/questions/19997989/periodic-task-for-celery-sent-but-not-executed
  [21]: http://stackoverflow.com/questions/19054008/how-to-use-environment-variables-with-supervisor-gunicorn-and-django-1-6