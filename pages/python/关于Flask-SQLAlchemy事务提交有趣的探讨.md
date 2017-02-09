location:python/关于Flask-SQLAlchemy事务提交有趣的探讨
modifyAt:2016-12-15 22:10:49
author:xbynet
createAt:2016-12-15 22:10:49
title:关于Flask-SQLAlchemy事务提交有趣的探讨

最近在开发[mdwiki][1]的时候遇到这样一个问题.Post is unbond to session.
我就好奇了

    post=Post.query.filter_by(location=location).first()
    abspath=util.getAbsPostPath(post.location)
    tagsList=[]
    ...
    print(post in session) #False
    post.tags=tagsList
    
这样还报post不在session中的错？没有显示调用db.session.commit()啊.
加一行测试：
print(post in session) #False
无奈，一个一个翻post=Post.query.filter_by(location=location).first()到post.tags=tagsList之间调用的每一个函数，终于在util.getAbsPostPath找到可疑点

    def getAbsPostPath(location):
        with current_app.app_context():              
            abspath=os.path.join(current_app.config['PAGE_DIR'],location.replace('/',os.sep))+".md"
        return abspath

但是这里也没有显式提交。只是多push了一个app_context，也不至于这样吧？
无奈之下查看Flask-SQLAlchemy源码，还好这货只有两个文件，比较少。
有这么一段：

```
        # 0.9 and later
        if hasattr(app, 'teardown_appcontext'):
            teardown = app.teardown_appcontext
        # 0.7 to 0.8
        elif hasattr(app, 'teardown_request'):
            teardown = app.teardown_request
        # Older Flask versions
        else:
            if app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN']:
                raise RuntimeError("Commit on teardown requires Flask >= 0.7")
            teardown = app.after_request

        @teardown
        def shutdown_session(response_or_exc):
            if app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN']:
                if response_or_exc is None:
                    self.session.commit()
            self.session.remove()
            return response_or_exc
```
这下明白了，原理是它监听了app.teardown_appcontext事件，在该事件发生时会调用
self.session.remove()移除session。这样一来把这一行注释掉，直接使用config模块就解决问题了

    def getAbsPostPath(location):
           # with current_app.app_context(): 
           abspath=os.path.join(config.PAGE_DIR,location.replace('/',os.sep))+".md"  

但同时看到了这个选项SQLALCHEMY_COMMIT_ON_TEARDOWN，是不是Flask-SQLAlchemy可以配置请求执行完逻辑之后自动提交，而不用我们每次都手动调用session.commit()?通过源码看答案是肯定的。
但是好奇的我还是google之，然后在github上看到了这样几段有趣的讨论：先贴地址
https://github.com/mitsuhiko/flask-sqlalchemy/issues/262
https://github.com/mitsuhiko/flask-sqlalchemy/issues/216
https://github.com/rosariomgomez/tradyfit/issues/34

然后在[官网][2]看到这样一段：

> Consider SQLALCHEMY_COMMIT_ON_TEARDOWN harmful and remove from docs.

**什么？考虑移除这一特性？**
刚刚知道这么方便的特性，准备用来着，就要被移除？更何况源码中也没有提示要移除啊。
其实是这样的，作者准备在3.0版本移除SQLALCHEMY_COMMIT_ON_TEARDOWN这一特性，目前自2.1以后从文档中移除了相关介绍。

为什么？接下来总结下大神们的探讨。

> mattupstate commented on 31 Jan 2015： I'd guess that the reason is due
> to the teardown_appcontext callback carrying a bug that, even if you
> catch an exception during the app context, the response_or_exc will
> never be None. In other words, teardown_appcontext suffers from a
> general Python exception handling bug.

这位mattupstate说teardown_appcontext回调存在一个bug，就是即使你正确地捕获了所有的bug，但是回调函数的第一个参数response_or_exc仍然不会为None。这一点令人费解。于是我试验了一发，包括没有bug的情形，主动抛出并捕获的情形，以及after_request中捕获并抛出的情形，都发现response_or_exc为None，没有重现他所说的。why?好想知道为什么。猜测可能是我Python版本，Flask版本的关系？继续看吧

> immunda commented on 3 Feb 2015 Sorry for the silence on this. Yep,
> that's the motivation, moving away from the (flawed) magic. I'm
> waiting to deprecate it entirely (3.0), because there's plans to
> introduce a more explicit transaction decorator first.

我去，连Flask-SQLAlchemy作者都支持这一观点，好吧，虽然我没有重现该问题，但是还是就这么认为吧，不用这个特性了。但是还是好奇地看了一下其他的观点。

原来实际上问题是这样的，见https://github.com/mitsuhiko/flask-sqlalchemy/issues/262

先贴上FLask-SQLAlchemy那部分代码：

```
       @teardown
        def shutdown_session(response_or_exc):
            if app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN']:
                if response_or_exc is None:
                    self.session.commit()
            self.session.remove()
            return response_or_exc
```
如果在app.teardown_request中或者在self.session.commit()时发生异常，而这个异常在这里并没有被捕获，那么self.session.remove()也就没有执行，那么这就会影响到下一个请求，下一个请求获取到的session其实是上一个带回滚状态的session，从而导致请求没有按预期效果执行而失败。至此问题算明白了。**并不是mattupstate这哥们形容的那样。那么这应该是flask实现机制导致的吧。**继续挖。
https://github.com/pallets/flask/pull/1451
http://stackoverflow.com/questions/23301968/invalid-transaction-persisting-across-requests
这哥们garaden给Flask提交了代码合并请求，关键部分如下
```
+    def wrap_teardown_func(teardown_func):
 +        @wraps(teardown_func)
 +        def log_teardown_error(*args, **kwargs):
 +            try:
 +                teardown_func(*args, **kwargs)
 +            except Exception as exc:
 +                app.logger.exception(exc)
 +        return log_teardown_error
 +
 +    if app.teardown_request_funcs:
 +        for bp, func_list in app.teardown_request_funcs.items():
 +            for i, func in enumerate(func_list):
 +                app.teardown_request_funcs[bp][i] = wrap_teardown_func(func)
 +    if app.teardown_appcontext_funcs:
 +        for i, func in enumerate(app.teardown_appcontext_funcs):
 +            app.teardown_appcontext_funcs[i] = wrap_teardown_func(func)
```
如果合并了这部分代码之后，那么以后注册app.teardown_request和app.teardown_appcontext，时异常将会自动被捕获。这在https://github.com/pallets/flask/pull/1822可以看到新版本Flask已经合并了这部分代码，不存在该问题了。
后面讨论看到

> Recently PR pallets/flask#1822 got merged into Flask. Will this maybe
> change the fact whether SQLALCHEMY_COMMIT_ON_TEARDOWN will still be
> removed in future?

**但这对于解决FLask-SQLAlchemy中的问题好像还是没有帮助？是不是我理解错了？如果session.commit发生异常，session.remove这样还是不会执行?**
后面看了https://github.com/pallets/flask/pull/1822中的代码，优化了application context从栈中pop的逻辑，这次的代码提交确保了tear_down回调处理发生异常时不会导致application context无法从栈中弹出而影响后续请求。这下大致明白了。Flask-SQLAlchemy中的db.session依赖于Application Context,所以如果这次Flask能确保无论如何最后会正确弹出application context，那么db.session也随之销毁了，那就不存在后续的影响了。但是，最后这句话我也不敢保证，只能是猜想。


**所以，言归正传，如果不用SQLALCHEMY_COMMIT_ON_TEARDOWN这一特性，那么我们怎么确保每次自动提交session呢？**
第一种：不是自动，全手动模式commit()，看讨论还是有很多人喜欢这种方式的，不过我讨厌每次都调用commit()
第二种：在after_request中进行提交commit，在teardown_request进行remove
虽说Flask已经修正不需要捕获也可以，但是为了编码的优雅(暂时找不到好点的词)，还是在dbsession_clean中进行了异常捕获。
```
@app.after_request
def after_clean(resp,*args,**kwargs):
    db.session.commit()
    return resp
@app.teardown_request
def dbsession_clean(exception=None):
    try:
        db.session.remove()
    finally:
        pass
```

第三种：使用自定义装饰器

```
def route(app_or_sub,rule,**options):
    def decorator(f):
        @wraps(f)
        def decorated_view(*args,**kwargs):
            res=f(*args,**kwargs)
            db.session.commit()
            return res
        endpoint = options.pop('endpoint', None)
        app_or_sub.add_url_rule(rule, endpoint, decorated_view, **options)
        return decorated_view
    return decorator
```


  [1]: https://github.com/xbynet/mdwiki
  [2]: http://flask-sqlalchemy.pocoo.org/2.1/changelog/