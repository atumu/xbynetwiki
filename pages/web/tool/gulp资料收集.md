author:xbynet
title:gulp资料收集
modifyAt:2016-12-12 11:24:46
location:web/tool/gulp资料收集
createAt:2016-12-12 11:24:46

原文：https://github.com/Platform-CUF/use-gulp
# use-gulp

#### 为什么使用gulp?
首先看一篇文章 [Gulp的目标是取代Grunt](http://www.infoq.com/cn/news/2014/02/gulp)
>根据gulp的文档，它努力实现的主要特性是：
>   - 易于使用：采用代码优于配置策略，gulp让简单的事情继续简单，复杂的任务变得可管理。
>   - 高效：通过利用node.js强大的流，不需要往磁盘写中间文件，可以更快地完成构建。
>   - 高质量：gulp严格的插件指导方针，确保插件简单并且按你期望的方式工作。
>   - 易于学习：通过把API降到最少，你能在很短的时间内学会gulp。构建工作就像你设想的一样：是一系列流管道。

> Gulp通过**流和代码优于配置**策略来尽量简化任务编写的工作。

别的先不说，通过代码来比较两者（gulp VS grunt）
可以参照我的代码，也可以阅读[该文章] (http://www.techug.com/gulp)。

- [Gruntfile.js](https://github.com/hjzheng/angular-cuf-nav/blob/master/Gruntfile.js)
- [gulpfile.js](https://github.com/hjzheng/html2js-gulp-for-cuf/blob/master/gulpfile.js)

两者的功能基本类似，gulp是通过代码完成任务，体现了代码优于配置的原则，对程序员更加友好，另外它也可以将多个功能一次性串起来，不需要暂存在本地，体现了对流的使用，这个可以阅读[该文章](http://www.techug.com/gulp)里的例子。

另外，经常会有人问，为什么gulp比grunt快，这个可以参考这篇[文章](http://jaysoo.ca/2014/01/27/gruntjs-vs-gulpjs/) 或者我本人在segmentfault上得回答[编译同样的scss，为什么gulp的速度几乎是grunt的两倍?](http://segmentfault.com/q/1010000003951849/a-1020000003952258)

#### 关于NodeJS流(stream)
因为gulp是基于流的方式工作的，所以想要进一步深入gulp，我们应该先学习NodeJS的流, 当然即使对流不熟悉，依然可以很方便的使用gulp。
  - 资料
    - [NodeSchool stream-adventure](https://github.com/substack/stream-adventure)
    - [stream-handbook](https://github.com/substack/stream-handbook)
  - 相关视频
    - [How streams help to raise Node.js performance](https://www.youtube.com/watch?v=QgEuZ52OZtU&list=PLPlAdM3UjHKok9rS8_RTQTSLtRBThk1ni&index=2)
    - [Node.js streams for the utterly confused](https://www.youtube.com/watch?v=9llfAByho98&index=1&list=PLPlAdM3UjHKok9rS8_RTQTSLtRBThk1ni)

#### 常用资料
- Gulp官网 http://gulpjs.com/
- Gulp中文网 http://www.gulpjs.com.cn/
- Gulp中文文档 https://github.com/lisposter/gulp-docs-zh-cn
- Gulp插件网 http://gulpjs.com/plugins/
- Awesome Gulp https://github.com/alferov/awesome-gulp
- StuQ-Gulp实战和原理解析 http://i5ting.github.io/stuq-gulp/
- glob用法 https://github.com/isaacs/node-glob


#### gulp常用插件

- **流控制**
  - [event-stream](http://www.atticuswhite.com/blog/merging-gulpjs-streams/) 事件流，不是插件但很有用 
  - [gulp-if](https://github.com/robrich/gulp-if) 有条件的运行一个task
  - [gulp-clone](https://github.com/mariocasciaro/gulp-clone) Clone files in memory in a gulp stream 非常有用
  - [vinyl-source-stream](https://github.com/hughsk/vinyl-source-stream) Use conventional text streams at the start of your gulp or vinyl pipelines 

- **AngularJS**
  - [gulp-ng-annotate](https://github.com/Kagami/gulp-ng-annotate) 注明依赖 for angular
  - [gulp-ng-html2js](https://github.com/marklagendijk/gulp-ng-html2js) html2js for angular
  - [gulp-angular-extender](https://libraries.io/npm/gulp-angular-extender) 为angular module添加dependencies
  - [gulp-angular-templatecache](https://github.com/miickel/gulp-angular-templatecache) 将html模板缓存到$templateCache中

- **文件操作**
  - [gulp-clean](https://github.com/peter-vilja/gulp-clean)  删除文件和目录, 请用[del](https://github.com/sindresorhus/del)来代替它[Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-11-10)
  - [gulp-concat](https://github.com/wearefractal/gulp-concat) 合并文件
  - [gulp-rename](https://github.com/hparra/gulp-rename) 重命名文件
  - [gulp-order](https://github.com/sirlantis/gulp-order) 对src中的文件按照指定顺序进行排序
  - [gulp-filter](https://github.com/sindresorhus/gulp-filter) 过滤文件 非常有用 [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/blob/master/2015-11-10/gulpfile.js)
  - [gulp-flatten](https://github.com/armed/gulp-flatten) 当拷贝文件时，不想拷贝目录时使用 [segmentfault](http://segmentfault.com/q/1010000004266922)

- **压缩**
  - [gulp-minify-css](https://github.com/murphydanger/gulp-minify-css)压缩css
  - [gulp-uglify](https://github.com/terinjokes/gulp-uglify) 用uglify压缩js
  - [gulp-imagemin](https://github.com/sindresorhus/gulp-imagemin) 压缩图片
  - [gulp-minify-html](https://github.com/murphydanger/gulp-minify-html) 压缩html
  - [gulp-csso](https://github.com/ben-eb/gulp-csso) 优化CSS


- **工具**
  - [gulp-load-plugins](https://github.com/jackfranklin/gulp-load-plugins) 自动导入gulp plugin
  - [gulp-load-utils](https://www.npmjs.com/package/gulp-load-utils) 增强版gulp-utils
  - [gulp-task-listing](https://github.com/OverZealous/gulp-task-listing) 快速显示gulp task列表
  - [gulp-help](https://github.com/chmontgomery/gulp-help) 为task添加帮助描述
  - [gulp-jsdoc](https://github.com/jsBoot/gulp-jsdoc) 生成JS文档
  - [gulp-plumber](https://github.com/floatdrop/gulp-plumber) Prevent pipe breaking caused by errors from gulp plugins [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-11-10)
  - [yargs](https://github.com/bcoe/yargs) 处理 process.argv
  - [run-sequence](https://github.com/OverZealous/run-sequence) 顺序执行 gulp task，gulp 4.0 已经支持该功能 `gulp.series(...tasks)`
  - [gulp-notify](https://github.com/mikaelbr/gulp-notify) gulp plugin to send messages based on Vinyl Files
  - [gulp-shell](https://github.com/sun-zheng-an/gulp-shell) 非常有用
  - [gulp-grunt](https://github.com/gratimax/gulp-grunt) 在gulp中运行grunt task

- **JS/CSS自动注入**
  - [gulp-usemin](https://github.com/zont/gulp-usemin) Replaces references to non-optimized scripts or stylesheets into a set of HTML files
  - [gulp-inject](https://github.com/klei/gulp-inject) 在HTML中自动添加style和script标签 [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-8-17/bower-dependence-inject)
  - [wiredep](https://github.com/taptapship/wiredep) 将bower依赖自动写到index.html中 [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-8-17/bower-dependence-inject)
  - [gulp-useref](https://github.com/jonkemp/gulp-useref) 功能类似与usemin [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-8-17/bower-dependence-inject) 新版本用法有变化，详见gulp-useref的README.md

- **代码同步**
  - [browser-sync](https://github.com/BrowserSync/browser-sync) 自动同步浏览器，结合gulp.watch方法一起使用 [Example 1](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-5-30/gulp-babel-test) [Example 2](https://github.com/hjzheng/es6-practice/blob/master/gulpfile.js)
  - [gulp-nodemon](https://github.com/JacksonGariety/gulp-nodemon) server端代码同步

- **Transpilation**
  - [gulp-babel](https://github.com/babel/gulp-babel) 将ES6代码编译成ES5   [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-5-30/gulp-babel-test)
  - [babelify](https://github.com/babel/babelify)  Browserify transform for Babel
  - [gulp-traceur](https://github.com/sindresorhus/gulp-traceur)  Traceur is a JavaScript.next-to-JavaScript-of-today compiler 

- **打包**
  - [gulp-browserify](https://www.npmjs.com/package/gulp-browserify)  用它和 babelify 实现ES6 module [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-5-30/gulp-es6-module)

- **编译**
  - [gulp-less](https://github.com/plus3network/gulp-less)  处理less [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-7-23/gulp-less-bootstrap)
  - [gulp-sass](https://github.com/dlmanning/gulp-sass) 处理sass

- **代码分析**
  - [gulp-jshint](https://github.com/spalger/gulp-jshint) JSHint检查 [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-11-10)
  - [gulp-jscs](https://github.com/jscs-dev/gulp-jscs) 检查JS代码风格 [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-11-10)

- **特别推荐**
  - [gulp-changed](https://github.com/sindresorhus/gulp-changed) 只传输修改过的文件
  - [gulp-cached](https://github.com/wearefractal/gulp-cached) 将文件先cache起来，先不进行操作
  - [gulp-remember](https://github.com/ahaurw01/gulp-remember) 和gulp-cached一块使用
  - [gulp-newer](https://github.com/tschaub/gulp-newer) pass through newer source files only, supports many:1 source:dest

- **其他**
  - [webpack-stream](https://github.com/shama/webpack-stream) gulp与webpack [Example](https://github.com/hjzheng/angular-es6-practice/blob/master/gulp/scripts.js)
  - [gulp-autoprefixer](https://github.com/sindresorhus/gulp-autoprefixer)  Prefix CSS
  - [gulp-sourcemaps](https://github.com/floridoo/gulp-sourcemaps) 生成source map文件
  - [gulp-rev](https://github.com/sindresorhus/gulp-rev) Static asset revisioning by appending content hash to filenames: unicorn.css → unicorn-d41d8cd98f.css 
  - [gulp-rev-replace](https://github.com/jamesknelson/gulp-rev-replace) [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-11-10)
  - [gulp-iconfont](https://github.com/nfroidure/gulp-iconfont) 制作iconfont [Example](https://github.com/hjzheng/CUF_meeting_knowledge_share/tree/master/2015-7-24/gulp-test-iconfont)
  - [gulp-svg-symbols](https://github.com/Hiswe/gulp-svg-symbols) 制作SVG Symbols, [关于使用SVG Symbol](http://isux.tencent.com/zh-hans/16292.html)
  - [gulp-template](https://github.com/sindresorhus/gulp-template) 模板替换
  - [gulp-dom-src](https://github.com/cgross/gulp-dom-src) 将html中的script，link等标签中的文件转成gulp stream。
  - [gulp-cheerio](https://github.com/KenPowers/gulp-cheerio) Manipulate HTML and XML files with Cheerio in Gulp. 
  - [require-dir](https://www.npmjs.com/package/require-dir) 利用它我们可以将 gulpfile.js 分成多个文件，具体用法可以参考这个[Splitting a gulpfile into multiple files](http://macr.ae/article/splitting-gulpfile-multiple-files.html)
  - [gulp-nodemon](https://github.com/JacksonGariety/gulp-nodemon) 强烈推荐, 监控你的node应用,并重现启动server

#### gulp入门视频 

- **Learning Gulp** (youtube)
  - [Learning Gulp #1 - Installing & Introducing Gulp ](https://www.youtube.com/watch?v=wNlEK8qrb0M)
  - [Learning Gulp #2 - Using Plugins & Minifying JavaScript](https://www.youtube.com/watch?v=Kh4eYdd8O4w)
  - [Learning Gulp #3 - Named Tasks ](https://www.youtube.com/watch?v=YBGeJnMrzzE)
  - [Learning Gulp #4 - Watching Files With Gulp ](https://www.youtube.com/watch?v=0luuGcoLnxM)
  - [Learning Gulp #5 - Compiling Sass With Gulp ](https://www.youtube.com/watch?v=cg7lwX0u-U0)
  - [Learning Gulp #6 - Keep Gulp Running With Plumber ](https://www.youtube.com/watch?v=rF6niaDKcxE)
  - [Learning Gulp #7 - Handling Errors Without Plumber ](https://www.youtube.com/watch?v=o24f4imRbxQ)
  - [Learning Gulp #8 - LiveReload With Gulp ](https://www.youtube.com/watch?v=r5fvdIa0ETk)
  - [Learning Gulp #9 - Easy Image Compression](https://www.youtube.com/watch?v=oXxMdT7T9qU)
  - [Learning Gulp #10 - Automatic Browser Prefixing ](https://www.youtube.com/watch?v=v259QplNDKk)
  - [Learning Gulp #11 - Gulp Resources & What's Next ](https://www.youtube.com/watch?v=vGCzovUFBIY)

- **Get started with gulp**(youtube)
  - [Get started with gulp Part 1: Workflow overview and jade templates](https://www.youtube.com/watch?v=DkRoa2LooNM&index=8&list=PLhIIfyPeWUjoySSdufaqfaSLeQDmCCY3Q)
  - [Get started with gulp Part 2: gulp & Browserify](https://www.youtube.com/watch?v=78_iyqT-qT8&index=9&list=PLhIIfyPeWUjoySSdufaqfaSLeQDmCCY3Q)
  - [Get started with gulp Part 3: Uglify & environment variables](https://www.youtube.com/watch?v=gRzCAyNrPV8&index=10&list=PLhIIfyPeWUjoySSdufaqfaSLeQDmCCY3Q)
  - [Get started with gulp Part 4: SASS & CSS minification](https://www.youtube.com/watch?v=O_0S6Z9FIKM&index=11&list=PLhIIfyPeWUjoySSdufaqfaSLeQDmCCY3Q)
  - [Get started with gulp Part 5: gulp.watch for true automation](https://www.youtube.com/watch?v=nsMsFyLGjSA&list=PLhIIfyPeWUjoySSdufaqfaSLeQDmCCY3Q&index=12)
  - [Get started with gulp Part 6: LiveReload and web server](https://www.youtube.com/watch?v=KURMrW-HsY4&list=PLhIIfyPeWUjoySSdufaqfaSLeQDmCCY3Q&index=13)

- **Gulp in Action** (慕课网)
  - [Gulp in Action(一)](http://www.imooc.com/video/5692)
  - [Gulp in Action(二)](http://www.imooc.com/video/5693)
  - [Gulp in Action(三)](http://www.imooc.com/video/5694)

- **BGTSD** (youtube)
  - [BGTSD - Part 20: Gulp and Babel Basics ](https://www.youtube.com/watch?v=Mo2xqBPbnlQ)
  - [BGTSD - Part 21: TypeScript and Gulp Basics ](https://www.youtube.com/watch?v=5Z82cpVP_qo)

- **John Papa**(付费)
  - [JavaScript Build Automation With Gulp.js](http://www.pluralsight.com/courses/javascript-build-automation-gulpjs)

#### gulp精彩文章
- [Using GulpJS to Generate Environment Configuration Modules](http://www.atticuswhite.com/blog/angularjs-configuration-with-gulpjs/)
- [Introduction to Gulp.js](http://stefanimhoff.de/2014/gulp-tutorial-1-intro-setup/)
- [Merging multiple GulpJS streams into one output file](http://www.atticuswhite.com/blog/merging-gulpjs-streams/)
- [Getting ES6 modules working thanks to Browserify, Babel, and Gulp](http://advantcomp.com/blog/ES6Modules/)
- Gulp学习指南系列：
  - [Gulp学习指南之入门](http://segmentfault.com/a/1190000002768534)
  - [Gulp学习指南之CSS合并、压缩与MD5命名及路径替换](http://segmentfault.com/a/1190000002932998)
- [6 Gulp Best Practices](http://blog.rangle.io/angular-gulp-bestpractices/?utm_source=javascriptweekly&utm_medium=email) :star:
  - Automate all Imports (gulp-inject, wiredep, useref and angular-file-sort)
  - Understand directory structure requirements 
  - Provide distinct development and production builds  (browser-sync)
  - Inject files with gulp-inject and wiredep ( gulp-inject and wiredep )
  - Create production builds with gulp-useref (gulp-useref)
  - Separate Gulp tasks into multiple files ```require('require-dir')('./gulp')```
- [Gulp 范儿 -- Gulp 高级技巧](http://csspod.com/advanced-tips-for-using-gulp-js/) :star:
- [Gulp 错误管理](http://csspod.com/error-management-in-gulp/) :star:
- [探究Gulp的Stream](http://segmentfault.com/a/1190000003770541) :star:
- [Gulp安装及配合组件构建前端开发一体化](http://www.dbpoo.com/getting-started-with-gulp/)
- [Gulp入门指南](https://github.com/nimojs/gulp-book)
- [Gulp入门指南 - nimojs](https://github.com/nimojs/blog/issues/19)
- [Gulp入门教程](http://markpop.github.io/2014/09/17/Gulp%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/)
- [Gulp开发教程（翻译）](http://www.w3ctech.com/topic/134)
- [Gulp：任务自动管理工具 - ruanyifeng](http://javascript.ruanyifeng.com/tool/gulp.html)
- [BrowserSync — 你值得拥有的前端同步测试工具](http://segmentfault.com/a/1190000003787713)
- [Essential Plugins for Gulp](http://ipestov.com/essential-plugins-for-gulp/) :star:
- [10 things to know about Gulp](http://engineroom.teamwork.com/10-things-to-know-about-gulp/?utm_source=javascriptweekly&utm_medium=email) :star:
- [Writing a gulp plugin](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/README.md) :star:
- [Gulp Plugin 开发](https://segmentfault.com/a/1190000000704549) :star:
- [前端 | 重构 gulpfile.js](https://segmentfault.com/a/1190000002880177)
- [gulp使用经验谈](http://www.qiqiboy.com/post/61)
- [Splitting a gulpfile into multiple files](http://macr.ae/article/splitting-gulpfile-multiple-files.html) :star:
- [Make your Gulp modular](http://makina-corpus.com/blog/metier/2015/make-your-gulp-modular)
- [gulp 传参数 实现定制化执行任务](http://yijiebuyi.com/blog/d64c5d28eb539941bf3b855d333850cc.html) 使用 `gulp.env`

#### gulp和ES6
- [在gulp中使用ES6](http://segmentfault.com/a/1190000004136053) :star:
- [Using ES6 with gulp](https://markgoodyear.com/2015/06/using-es6-with-gulp/)

#### gulp和babelify
- [Example](https://gist.github.com/hjzheng/0ff59d37aa23fbd831e081138c6f24f9)

#### debug gulp task
- [Debugging Gulp.js Tasks](http://www.greg5green.com/blog/debugging-gulp-js-tasks/)
- [Debug command line apps like Gulp](https://github.com/s-a/iron-node/blob/master/docs/DEBUG-NODEJS-COMMANDLINE-APPS.md)

#### gulp项目结构应用实例
- [gulp-AngularJS1.x-seed](https://github.com/hjzheng/gulp-AngularJS1.x-seed) :star: 自己写的一个开发环境(gulp + AngularJS1.x)
- [gulp模式](https://github.com/johnpapa/gulp-patterns) 
- [gf-skeleton-angularjs](https://github.com/gf-rd/gf-skeleton-angularjs)
- [generator-hottowel](https://github.com/johnpapa/generator-hottowel)
- [generator-gulp-angular](https://github.com/swiip/generator-gulp-angular#readme)
- [generator-gulper](https://github.com/leaky/generator-gulper)

#### gulp常见问题 [segmentfault之gulp](http://segmentfault.com/t/gulp?type=newest)

- [如何拷贝一个目录?](http://stackoverflow.com/questions/25038014/how-do-i-copy-directories-recursively-with-gulp)
```js
gulp.src([ files ], { "base" : "." })
```

#### gulp 4.0 相关
目前 gulp 4.0 还没有正式release，先推荐几篇文章让大家热热身。

- [Gulp 4.0 前瞻](http://segmentfault.com/a/1190000002528547)
- [Gulp 4.0 API 文档](https://github.com/cssmagic/blog/issues/55)
- [是时候升级你的gulp到4.0了](http://www.alloyteam.com/2015/07/update-your-gulp/)
- [Migrating to gulp 4 by example](https://blog.wearewizards.io/migrating-to-gulp-4-by-example)

不定期更新中 ... ...
