title: jquery页面加载进度条插件 

#  jQuery页面加载进度条插件nprogress 
![](/data/dokuwiki/web/pasted/20160505-163255.png)
github:https://github.com/rstacruz/nprogress
demo:http://ricostacruz.com/nprogress
Slim progress bars for Ajax'y applications. Inspired by Google, YouTube, and Medium.
安装：
```

<script src='nprogress.js'></script>
<link rel='stylesheet' href='nprogress.css'/>

```
基本使用：
```

NProgress.start();
NProgress.done();

```
高级用法：
1、设定进度百分比
```

NProgress.set(0.0);     // Sorta same as .start()
NProgress.set(0.4);
NProgress.set(1.0);     // Sorta same as .done()

```

2、增加进度值
NProgress.inc(0.2);

3、不定进度：
NProgress.inc();

4、获取状态值:
NProgress.status

5、强制完成:
NProgress.done(true);

配置
1、起始进度：
```

NProgress.configure({ minimum: 0.1 });

```
2、模板
3、速度和动画效果：
```

NProgress.configure({ easing: 'ease', speed: 500 });

```