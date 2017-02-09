title: gulp前端自动化构建工具 

#  gulp前端自动化构建工具 
官网:http://gulpjs.com/
github与文档:https://github.com/gulpjs/gulp/tree/master/docs
中文:http://www.gulpjs.com.cn/docs/#articles/
部分插件中文说明:http://www.ydcss.com/archives/tag/gulp
参考：
http://markpop.github.io/2014/09/17/Gulp%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/
http://www.ydcss.com/archives/18
http://blog.jobbole.com/80338/

npm插件搜索：https://www.npmjs.com/browse/keyword/gulpplugin
gulp插件搜索:http://gulpjs.com/plugins/

gulp资料大全：https://segmentfault.com/a/1190000004915222

**gulp**是最近流行前端构建工具，苦于之前使用Grunt，代码很难阅读，现在出了Gulp，真是摆脱了痛苦。
**为什么使用Gulp**
Gulp基于Node.js的前端构建工具，通过Gulp的插件可以**实现前端代码的编译（sass、less）、压缩、测试；图片的压缩；浏览器自动刷新**，还有许多强大的插件可以在[这里](http://gulpjs.com/plugins/)查找。比起Grunt不仅配置简单而且更容易阅读和维护。
```

gulp.task('sass', function() {
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'compressed' }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/assets/css'))
});

```
**使用Gulp我们只需要放一个路径，通过管道方式使用插件，最后生成文件**，是不是有种jQuery的感觉。现在是否对Gulp感兴趣了，那就开始使用Gulp吧！

##  安装 
安装nodejs -> 全局安装gulp -> 项目安装gulp以及gulp插件 -> 配置gulpfile.js -> 运行任务
1、首先安装nodejs。略
3.1、说明：npm（node package manager）nodejs的包管理器，用于node插件管理（包括安装、卸载、管理依赖等）；
使用npm安装插件：命令提示符执行npm install <name> [-g] [--save-dev]；
<name>：node插件名称。例：npm install gulp-less --save-dev
  * -g：全局安装。将会安装在C:\Users\Administrator\AppData\Roaming\npm，**并且写入系统环境变量**；**非全局安装：将会安装在当前定位目录；  全局安装可以通过命令行在任何地方调用它，本地安装将安装在定位目录的node_modules文件夹下，通过nodejs的require()调用；**
  * --save：将保存配置信息至` package.json `（**package.json是nodejs项目配置文件**）；
  * -dev：保存至package.json的devDependencies节点，不指定-dev将保存至dependencies节点；
**为什么要保存至package.json？**
因为node插件包相对来说非常庞大，所以加入版本管理，将配置信息写入package.json并将其加入版本管理，其他开发者对应下载即可
（命令提示符**执行npm install，则会根据package.json下载所有需要的包**）。

使用npm卸载插件：npm uninstall <name> [-g] [--save-dev]  PS：不要直接删除本地插件包
使用npm更新插件：npm update <name> [-g] [--save-dev]
查看npm帮助：npm help
当前目录已安装插件：npm list
npm安装插件过程：从http://registry.npmjs.org下载对应的插件包（该网站服务器位于国外，所以经常下载缓慢或出现异常），解决办法往下看↓↓↓↓↓↓。

**npm使用国内镜像**
国内镜像官方网址：http://npm.taobao.org；
使用方式有四种：
1、安装官方封装的cnpm：(不推荐)
安装：命令提示符执行npm install cnpm -g --registry=https://registry.npm.taobao.org；或者直接使用镜像  
注意：安装完后最好查看其版本号cnpm -v或关闭命令提示符重写打开，安装完直接使用有可能会出现错误；
注：cnpm跟npm用法完全一致，只是在执行命令时将npm改为cnpm（以下操作将以cnpm代替npm）。

2、通过config命令
```

npm config set registry https://registry.npm.taobao.org

``` 

3、命令行指定
```

npm --registry https://registry.npm.taobao.org

``` 

4、编辑 ~/.npmrc 加入下面内容
registry = https://registry.npm.taobao.org

###  全局安装gulp 
安装：命令提示符执行npm install gulp -g；
查看是否正确安装：命令提示符执行gulp -v，出现版本号即为正确安装。
安装之后的位置:
C:\Users\xby\AppData\Roaming\npm\gulp -> C:\Users\xby\AppData\Roaming\npm\node_modules\gulp\bin\gulp.js
###  package.json文件(nodejs项目配置文件) 
说明：package.json是基于nodejs项目必不可少的配置文件，它是存放在项目根目录的普通json文件；
它是这样一个json文件（注意：json文件内是不能写注释的，复制下列内容请删除注释）：当然我们可以手动新建这个配置文件，
但是我们应该使用更为效率的方法：命令提示符执行` npm init `
```

{
  "name": "test",   //项目名称（必须）
  "version": "1.0.0",   //项目版本（必须）
  "description": "This is for study gulp project !",   //项目描述（必须）
  "homepage": "",   //项目主页
  "repository": {    //项目资源库
    "type": "git",
    "url": "https://git.oschina.net/xxxx"
  },
  "author": {    //项目作者信息
    "name": "surging",
    "email": "surging2@qq.com"
  },
  "license": "ISC",    //项目许可协议
  "devDependencies": {    //项目依赖的插件
    "gulp": "^3.8.11",
    "gulp-less": "^3.0.0"
  }
}

```
![](/data/dokuwiki/web/pasted/20160412-122606.png)
查看package.json帮助文档，命令提示符执行` npm help package.json `
特别注意：package.json是一个普通json文件，所以不能添加任何注释。参看 http://www.zhihu.com/question/23004511

###  项目本地安装gulp以及gulp插件 
安装：定位目录命令后提示符执行
```

npm install <name>  --save-dev；

1、首先得本地安装gulp：
cnpm install gulp --save-dev；
2、然后再安装gulp插件：
npm install gulp-less --save-dev

```
将会安装在` node_modules `的gulp-less目录下，该目录下有一个gulp-less的使用帮助文档README.md；
PS：细心的你可能会发现，我们全局安装了gulp，项目也安装了gulp，**全局安装gulp是为了执行gulp任务，本地安装gulp则是为了调用gulp插件的功能。**

**常用插件**
  * sass的编译（gulp-ruby-sass）
  * 自动添加css前缀（gulp-autoprefixer）
  * 压缩css（gulp-minify-css）
  * gulp-make-css-url-version  给css文件里面的url添加版本号
  * gulp-sourcemaps就是让gulp支持js sourcemap的能力
  * js代码校验（gulp-jshint）
  * 合并js文件（gulp-concat）
  * 压缩js代码（gulp-uglify）
  * 压缩图片（gulp-imagemin）
  * 自动刷新页面（gulp-livereload）
  * 图片缓存，只有图片替换了才压缩（gulp-cache）
  * 更改提醒（gulp-notify）
  * 清除文件（del）
安装这些插件需要运行如下命令：
```

$ npm install gulp gulp-ruby-sass gulp-make-css-url-version gulp-autoprefixer gulp-sourcemaps gulp-minify-css gulp-jshint gulp-concat gulp-uglify gulp-imagemin gulp-notify gulp-rename gulp-livereload gulp-cache del --save-dev

```

###  新建gulpfile.js文件（重要） 
说明：**gulpfile.js是gulp项目的配置文件**，是位于项目根目录的普通js文件（其实将gulpfile.js放入其他文件夹下亦可）。
它大概是这样一个js文件（更多插件配置请查看[这里](http://www.ydcss.com/archives/tag/gulp)）：
```

//导入工具包 require('node_modules里对应模块')
var gulp = require('gulp'), //本地安装gulp所用到的地方
    less = require('gulp-less');
 
//定义一个testLess任务（自定义任务名称）
gulp.task('testLess', function () {
    gulp.src('src/less/index.less') //该任务针对的文件
        .pipe(less()) //该任务调用的模块
        .pipe(gulp.dest('src/css')); //将会在src/css下生成index.css
});
 
gulp.task('default',['testLess', 'elseTask']); //定义默认任务
 
//gulp.task(name[, deps], fn) 定义任务  name：任务名称 deps：依赖任务名称 fn：回调函数
//gulp.src(globs[, options]) 执行任务处理的文件  globs：处理的文件路径(字符串或者字符串数组) 
//gulp.dest(path[, options]) 处理完后文件生成路径

```

####  加载插件 
接着我们要创建一个gulpfile.js在根目录下，然后在里面加载插件：
```

var gulp = require('gulp'),
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload'),
    del = require('del'),
    sourcemaps=require('gulp-sourcemaps');

```
####  建立任务 
编译sass、自动添加css前缀和压缩
首先我们编译sass，添加前缀，保存到我们指定的目录下面，还没结束，我们还要压缩，给文件添加.min后缀再输出压缩文件到指定目录，最后提醒任务完成了：
```

gulp.task('styles', function() {
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded' }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(rename({suffix: '.min'}))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(notify({ message: 'Styles task complete' }));
});

```
让我解释一下：
gulp.task('styles', function () {...});
gulp.task这个API用来创建任务，在命令行下可以输入$ gulp styles来执行上面的任务。
return gulp.src('src/styles/main.scss')
gulp.src这个API设置需要处理的文件的路径，可以是多个文件以数组的形式[main.scss, vender.scss]，也可以是正则表达式/* */*.scss。
.pipe(sass({ style: 'expanded' }))
我们使用.pipe()这个API将需要处理的文件导向sass插件，那些插件的用法可以在github上找到，为了方便大家查找我已经在上面列出来了。
.pipe(gulp.dest('dist/assets/css'));
gulp.dest()API设置生成文件的路径，一个任务可以有多个生成路径，一个可以输出未压缩的版本，另一个可以输出压缩后的版本。
为了更好的了解Gulp API，强烈建议看一下Gulp API文档，其实Gulp API就这么几个，没什么好可怕的。

####  js代码校验、合并和压缩 
希望大家已经知道如何去创建一个任务了，接下来我们完成scripts的校验、合并和压缩的任务吧：
```

gulp.task('scripts', function() {
  return gulp.src('src/scripts/**/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('default'))
    .pipe(concat('main.js'))
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(rename({suffix: '.min'}))
    .pipe(uglify())
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(notify({ message: 'Scripts task complete' }));
});

```
需要提醒的是我们要设置JSHint的reporter方式，上面使用的是default默认的，了解更多点击这里。

####  压缩图片 
```

gulp.task('images', function() {
  return gulp.src('src/images/**/*')
    .pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))
    .pipe(gulp.dest('dist/assets/img'))
    .pipe(notify({ message: 'Images task complete' }));
});

```
这个任务使用imagemin插件把所有在src/images/目录以及其子目录下的所有图片（文件）进行压缩，我们可以进一步优化，利用缓存保存已经压缩过的图片，使用之前装过的gulp-cache插件，不过要修改一下上面的代码：
将这行代码:
.pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))
修改成:
.pipe(cache(imagemin({ optimizationLevel: 5, progressive: true, interlaced: true })))
现在，只有新建或者修改过的图片才会被压缩了。

####  清除文件 
在任务执行前，最好先清除之前生成的文件：
```

gulp.task('clean', function(cb) {
    del(['dist/assets/css', 'dist/assets/js', 'dist/assets/img'], cb)
});

```
在这里没有必要使用Gulp插件了，可以使用NPM提供的插件。我们用一个回调函数（cb）确保在退出前完成任务。

####  设置默认任务（default） 
我们在命令行下输入$ gulp执行的就是默认任务，现在我们为默认任务指定执行上面写好的三个任务：
```

gulp.task('default', ['clean'], function() {
    gulp.start('styles', 'scripts', 'images');
});

```
在这个例子里面，clean任务执行完成了才会去运行其他的任务，**在gulp.start()里的任务执行的顺序是不确定的，所以将要在它们之前执行的任务写在数组里面。**

####  监听文件 
**为了监听文件的是否修改以便执行相应的任务**，我们需要创建一个新的任务，然后利用gulp.watchAPI实现：
```

gulp.task('watch', function() {
  // Watch .scss files
  gulp.watch('src/styles/**/*.scss', ['styles']);
  // Watch .js files
  gulp.watch('src/scripts/**/*.js', ['scripts']);
  // Watch image files
  gulp.watch('src/images/**/*', ['images']);
});

```
我们将不同类型的文件分开处理，执行对应的数组里的任务。现在我们可以在命令行输入` $ gulp watch `执行监听任务，当.sass、.js和图片修改时将执行对应的任务。

**自动刷新页面**
Gulp也可以实现当文件修改时自动刷新页面，我们要修改watch任务，配置LiveReload：
```

gulp.task('watch', function() {
  // Create LiveReload server
  livereload.listen();
  // Watch any files in dist/, reload on change
  gulp.watch(['dist/**']).on('change', livereload.changed);
});

```
要使这个能够工作，还需要在浏览器上安装LiveReload插件，除此之外还能这样做

** 所有任务放一起**
```

/*!
 * gulp
 * $ npm install gulp-ruby-sass gulp-autoprefixer gulp-minify-css gulp-jshint gulp-concat gulp-uglify gulp-imagemin gulp-notify gulp-rename gulp-livereload gulp-cache del --save-dev
 */
// Load plugins
var gulp = require('gulp'),
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload'),
    del = require('del');
// Styles
gulp.task('styles', function() {
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded', }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/styles'))
    .pipe(rename({ suffix: '.min' }))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/styles'))
    .pipe(notify({ message: 'Styles task complete' }));
});
// Scripts
gulp.task('scripts', function() {
  return gulp.src('src/scripts/**/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('default'))
    .pipe(concat('main.js'))
    .pipe(gulp.dest('dist/scripts'))
    .pipe(rename({ suffix: '.min' }))
    .pipe(uglify())
    .pipe(gulp.dest('dist/scripts'))
    .pipe(notify({ message: 'Scripts task complete' }));
});
// Images
gulp.task('images', function() {
  return gulp.src('src/images/**/*')
    .pipe(cache(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true })))
    .pipe(gulp.dest('dist/images'))
    .pipe(notify({ message: 'Images task complete' }));
});
// Clean
gulp.task('clean', function(cb) {
    del(['dist/assets/css', 'dist/assets/js', 'dist/assets/img'], cb)
});
// Default task
gulp.task('default', ['clean'], function() {
    gulp.start('styles', 'scripts', 'images');
});
// Watch
gulp.task('watch', function() {
  // Watch .scss files
  gulp.watch('src/styles/**/*.scss', ['styles']);
  // Watch .js files
  gulp.watch('src/scripts/**/*.js', ['scripts']);
  // Watch image files
  gulp.watch('src/images/**/*', ['images']);
  // Create LiveReload server
  livereload.listen();
  // Watch any files in dist/, reload on change
  gulp.watch(['dist/**']).on('change', livereload.changed);
});

```
在[gist](https://gist.github.com/markgoodyear/8497946#file-gruntfile-js)上有源码，并且附上Grunt的实现作为对比。
**运行gulp task**
说明：命令提示符执行gulp 任务名称；
编译less：命令提示符执行gulp testLess；
当执行gulp default或gulp将会调用default任务里的所有任务[‘testLess’,’elseTask’]。

##  总结 
  * 安装nodejs；
  * 新建package.json文件；
  * 全局和本地安装gulp；
  * 安装gulp插件；
  * 新建gulpfile.js文件；
  * 通过命令提示符运行gulp任务。

##  部分API说明 
http://www.gulpjs.com.cn/docs/api/
1、gulp.src(globs[, options])
参数:匹配模式（glob）或者匹配模式的数组（array of globs）
glob 请参考 [node-glob](https://github.com/isaacs/node-glob) 语法 或者，你也可以直接写文件的路径。
options配置选项中的一个有用配置base
```

gulp.src('client/js/**/*.js') // 匹配 'client/js/somedir/somefile.js' 并且将 `base` 解析为 `client/js/`
  .pipe(minify())
  .pipe(gulp.dest('build'));  // 写入 'build/somedir/somefile.js'

gulp.src('client/js/**/*.js', { base: 'client' })
  .pipe(minify())
  .pipe(gulp.dest('build'));  // 写入 'build/js/somedir/somefile.js'

```
###  glob模式语法说明 
基本语法介绍
  * * 匹配任意数量的字符，但不匹配/
  * ? 匹配单个字符，但不匹配/
  * * * 匹配任意数量的字符，包括/，只要它是路径中唯一的一部分（* *匹配的内容为路径的父目录或者为空时才会生效。）
  * {} 允许使用一个逗号分割的列表或者表达式
  * ! 在模式的开头用于否定一个匹配模式(即排除与模式匹配的信息)
ps：这里只是列了js版本的glob模式的一些常用语法。
2、gulp.dest(path[, options])
能被 pipe 进来，并且将会写文件。并且重新输出（emits）所有数据，因此**你可以将它 pipe 到多个文件夹。如果某文件夹不存在，将会自动创建它。**
```

gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));

```
**文件被写入的路径是以所给的相对路径根据所给的目标目录计算而来。类似的，相对路径也可以根据所给的 base 来计算。** 请查看上述的 gulp.src 来了解更多信息。

3、gulp.task(name[, deps], fn)
定义一个使用 Orchestrator 实现的任务（task）。
```

gulp.task('somename', function() {
  // 做一些事
});

```
name任务的名字，如果你需要在命令行中运行你的某些任务，那么，请不要在名字中使用空格。
deps类型： Array，**一个包含任务列表的数组，这些任务会在你当前任务运行之前完成。**

###  异步任务支持 
任务可以异步执行，如果 fn 能做到以下其中一点：
方式一、接受一个 callback参数
接受一个 callback，这样就能类似promise的方式确保异步任务完成之后执行某些操作。
```

// 在 shell 中执行一个命令
var exec = require('child_process').exec;
gulp.task('jekyll', function(cb) {//参数cb为callback，会被gulp自动传入进去。
  // 编译 Jekyll
  exec('jekyll build', function(err) {
    if (err) return cb(err); // 返回 error
    cb(); // 完成 task
  });
});

```
方式二、返回一个 stream
```

gulp.task('somename', function() {
  var stream = gulp.src('client/**/*.js')
    .pipe(minify())
    .pipe(gulp.dest('build'));
  return stream;
});

```
方式三、返回一个 promise
```

var Q = require('q');
gulp.task('somename', function() {
  var deferred = Q.defer();
  // 执行异步的操作
  setTimeout(function() {
    deferred.resolve();
  }, 1);

  return deferred.promise;
});

```
**示例**
```

var gulp = require('gulp');

// 返回一个 callback，因此系统可以知道它什么时候完成
gulp.task('one', function(cb) {
    // 做一些事 -- 异步的或者其他的
    cb(err); // 如果 err 不是 null 或 undefined，则会停止执行，且注意，这样代表执行失败了
});

// 定义一个所依赖的 task 必须在这个 task 执行之前完成
gulp.task('two', ['one'], function() {
    // 'one' 完成后
});

gulp.task('default', ['one', 'two']);

```

4、gulp.watch(glob [, opts], tasks) 或 gulp.watch(glob [, opts, cb])
监视文件，并且可以在文件发生改动时候做一些事情。它总会返回一个 EventEmitter 来发射（emit）`  change 事件。 `
```

var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
watcher.on('change', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});

```
```

gulp.watch('js/**/*.js', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});

```
callback 会被传入一个名为 event 的对象。这个对象描述了所监控到的变动：
event.type类型： String。发生的变动的类型：added, changed 或者 deleted。
event.path类型： String。触发了该事件的文件的路径。

##  JavaScript Source Map与gulp-sourcemaps的作用 
http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html
JavaScript脚本正变得越来越复杂。大部分源码（尤其是各种函数库和框架）都要经过转换，才能投入生产环境。
常见的源码转换，主要是以下三种情况：
　　（1）压缩，减小体积。比如jQuery 1.9的源码，压缩前是252KB，压缩后是32KB。
　　（2）多个文件合并，减少HTTP请求数。
　　（3）其他语言编译成JavaScript。最常见的例子就是CoffeeScript。
这三种情况，都使得实际运行的代码不同于开发代码，除错（debug）变得困难重重。
通常，JavaScript的解释器会告诉你，第几行第几列代码出错。但是，这对于转换后的代码毫无用处。举例来说，jQuery 1.9压缩后只有3行，每行3万个字符，所有内部变量都改了名字。你看着报错信息，感到毫无头绪，根本不知道它所对应的原始位置。
**这就是Source map想要解决的问题。**
二、什么是Source map
简单说，**Source map就是一个信息文件，里面储存着位置信息。也就是说，转换后的代码的每一个位置，所对应的转换前的位置。**
**有了它，出错的时候，除错工具将直接显示原始代码，而不是转换后的代码。**这无疑给开发者带来了很大方便。
目前，暂时只有Chrome浏览器支持这个功能。在Developer Tools的Setting设置中，确认选中"Enable source maps"。
![](/data/dokuwiki/web/pasted/20160412-133105.png)
三、如何启用Source map
只要在转换后的代码尾部，加上一行就可以了。
```

//@ sourceMappingURL=/path/to/file.js.map

```
map文件可以放在网络上，也可以放在本地文件系统。

gulp-sourcemaps就是让gulp支持这种能力，目前支持gulp-sourcemaps的gulp插件列表有https://github.com/floridoo/gulp-sourcemaps/wiki/Plugins-with-gulp-sourcemaps-support
使用教程:https://www.npmjs.com/package/gulp-sourcemaps
gulp-sourcemaps使用:
```

var gulp = require('gulp');
var concat = require('gulp-concat');
var sourcemaps = require('gulp-sourcemaps');
 
gulp.task('javascript', function() {
  return gulp.src('src/**/*.js')
    .pipe(sourcemaps.init())
      .pipe(concat('all.js'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('dist'));
});

```
##  gulp教程之gulp-uglify 
https://github.com/terinjokes/gulp-uglify#user-content-options
使用gulp-uglify压缩javascript文件，减小文件大小。
基本使用
```

var gulp = require('gulp'),
    uglify = require('gulp-uglify');
 
gulp.task('jsmin', function () {
    gulp.src('src/js/index.js')
        .pipe(uglify())
        .pipe(gulp.dest('dist/js'));
});

```
压缩多个js文件
```

var gulp = require('gulp'),
    uglify = require('gulp-uglify');
 
gulp.task('jsmin', function () {
    gulp.src(['src/js/index.js','src/js/detail.js']) //多个文件以数组形式传入
        .pipe(uglify())
        .pipe(gulp.dest('dist/js'));
});

```
匹配符“!”，“*”，“* *”，“{}”
```

var gulp = require('gulp'),
    uglify= require('gulp-uglify');
 
gulp.task('jsmin', function () {
    //压缩src/js目录下的所有js文件
    //除了test1.js和test2.js（**匹配src/js的0个或多个子文件夹）
    gulp.src(['src/js/*.js', '!src/js/**/{test1,test2}.js']) 
        .pipe(uglify())
        .pipe(gulp.dest('dist/js'));
});

```
指定变量名不混淆改变
```

var gulp = require('gulp'),
    uglify= require('gulp-uglify');
 
gulp.task('jsmin', function () {
    gulp.src(['src/js/*.js', '!src/js/**/{test1,test2}.js'])
        .pipe(uglify({
            //mangle: true,//类型：Boolean 默认：true 是否修改变量名
            mangle: {except: ['require' ,'exports' ,'module' ,'$']}//排除混淆关键字
        }))
        .pipe(gulp.dest('dist/js'));
});

```
```

var gulp = require('gulp'),
    uglify= require('gulp-uglify');
 
gulp.task('jsmin', function () {
    gulp.src(['src/js/*.js', '!src/js/**/{test1,test2}.js'])
        .pipe(uglify({
            mangle: true,//类型：Boolean 默认：true 是否修改变量名
            compress: true,//类型：Boolean 默认：true 是否完全压缩
            preserveComments: all //保留所有注释
        }))
        .pipe(gulp.dest('dist/js'));
});

```
##  gulp-autoprefixer插件说明 
http://www.ydcss.com/archives/94
使用gulp-autoprefixer根据设置浏览器版本自动处理浏览器前缀。使用她我们可以很潇洒地写代码，不必考虑各浏览器兼容前缀。
【特别是开发移动端页面时，就能充分体现它的优势。例如兼容性不太好的flex布局。】
gulp-autoprefixer的browsers参数详解 （[传送门](https://github.com/ai/browserslist#queries)）：
● last 2 versions: 主流浏览器的最新两个版本
● last 1 Chrome versions: 谷歌浏览器的最新版本
● last 2 Explorer versions: IE的最新两个版本
● last 3 Safari versions: 苹果浏览器最新三个版本
● Firefox >= 20: 火狐浏览器的版本大于或等于20
● iOS 7: IOS7版本
● Firefox ESR: 最新ESR版本的火狐
● > 5%: 全球统计有超过5%的使用率
3.3、发现上面规律了吗，相信这不难看出，接下来说说各浏览器的标识：
  * Android for Android WebView.
  * BlackBerry or bb for Blackberry browser.
  * Chrome for Google Chrome.
  * Firefox or ff for Mozilla Firefox.
  * Explorer or ie for Internet Explorer.
  * iOS or ios_saf for iOS Safari.
  * Opera for Opera.
  * Safari for desktop Safari.
  * OperaMobile or op_mob for Opera Mobile.
  * OperaMini or op_mini for Opera Mini.
  * ChromeAndroid or and_chr
  * FirefoxAndroid or and_ff for Firefox for Android.
  * ExplorerMobile or ie_mob for Internet Explorer Mobile.
```

var gulp = require('gulp'),
    autoprefixer = require('gulp-autoprefixer');
 
gulp.task('testAutoFx', function () {
    gulp.src('src/css/index.css')
        .pipe(autoprefixer({
            browsers: ['last 2 versions', 'Android >= 4.0'],
            cascade: true, //是否美化属性值 默认：true 像这样：
            //-webkit-transform: rotate(45deg);
            //        transform: rotate(45deg);
            remove:true //是否去掉不必要的前缀 默认：true 
        }))
        .pipe(gulp.dest('dist/css'));
});

```

##  gulp教程之gulp-minify-css插件说明 
http://www.ydcss.com/archives/41
基本使用:
```

var gulp = require('gulp'),
    cssmin = require('gulp-minify-css');
 
gulp.task('testCssmin', function () {
    gulp.src('src/css/*.css')
        .pipe(cssmin())
        .pipe(gulp.dest('dist/css'));
});

```
gulp-minify-css 最终是调用clean-css，其他参数查看[这里](https://github.com/jakubpawlowicz/clean-css#how-to-use-clean-css-api):
```

var gulp = require('gulp'),
    cssmin = require('gulp-minify-css');
 
gulp.task('testCssmin', function () {
    gulp.src('src/css/*.css')
        .pipe(cssmin({
            advanced: false,//类型：Boolean 默认：true [是否开启高级优化（合并选择器等）]
            compatibility: 'ie7',//保留ie7及以下兼容写法 类型：String 默认：''or'*' [启用兼容模式； 'ie7'：IE7兼容模式，'ie8'：IE8兼容模式，'*'：IE9+兼容模式]
            keepBreaks: true//类型：Boolean 默认：false [是否保留换行]
        }))
        .pipe(gulp.dest('dist/css'));
});

```
**给css文件里引用url加版本号（根据引用文件的md5生产版本号）,这样便可以清除上个版本的缓存影响**，像这样：
![](/data/dokuwiki/web/pasted/20160412-134510.png)
```

var gulp = require('gulp'),
    cssmin = require('gulp-minify-css');
    //确保已本地安装gulp-make-css-url-version [cnpm install gulp-make-css-url-version --save-dev]
    cssver = require('gulp-make-css-url-version'); 
 
gulp.task('testCssmin', function () {
    gulp.src('src/css/*.css')
        .pipe(cssver()) //给css文件里引用文件加版本号（文件MD5）
        .pipe(cssmin())
        .pipe(gulp.dest('dist/css'));
});

```

##  gulp教程之gulp-imagemin 
http://www.ydcss.com/archives/26
https://github.com/sindresorhus/gulp-imagemin#user-content-options
使用gulp-imagemin压缩图片文件（包括PNG、JPEG、GIF和SVG图片）
基本使用：
```

var gulp = require('gulp'),
    imagemin = require('gulp-imagemin');
 
gulp.task('testImagemin', function () {
    gulp.src('src/img/*.{png,jpg,gif,ico}')
        .pipe(imagemin())
        .pipe(gulp.dest('dist/img'));
});

```
```

var gulp = require('gulp'),
    imagemin = require('gulp-imagemin');
 
gulp.task('testImagemin', function () {
    gulp.src('src/img/*.{png,jpg,gif,ico}')
        .pipe(imagemin({
            optimizationLevel: 5, //类型：Number  默认：3  取值范围：0-7（优化等级）
            progressive: true, //类型：Boolean 默认：false 无损压缩jpg图片
            interlaced: true, //类型：Boolean 默认：false 隔行扫描gif进行渲染
            multipass: true //类型：Boolean 默认：false 多次优化svg直到完全优化
        }))
        .pipe(gulp.dest('dist/img'));
});

```
**只压缩修改的图片。压缩图片时比较耗时，在很多情况下我们只修改了某些图片，没有必要压缩所有图片，使用”gulp-cache”只压缩修改的图片，没有修改的图片直接从缓存文件读**取（C:Users\Administrator\AppData\Local\Temp\gulp-cache）。
```

var gulp = require('gulp'),
    imagemin = require('gulp-imagemin'),
    pngquant = require('imagemin-pngquant'),
    //确保本地已安装gulp-cache [cnpm install gulp-cache --save-dev]
    cache = require('gulp-cache');
    
gulp.task('testImagemin', function () {
    gulp.src('src/img/*.{png,jpg,gif,ico}')
        .pipe(cache(imagemin({
            progressive: true,
            svgoPlugins: [{removeViewBox: false}],
            use: [pngquant()]
        })))
        .pipe(gulp.dest('dist/img'));
});

```

##  gulp教程之gulp-concat合并文件 
http://www.ydcss.com/archives/83
```

var gulp = require('gulp'),
    concat = require('gulp-concat');
 
gulp.task('testConcat', function () {
    gulp.src('src/js/*.js')
        .pipe(concat('all.js'))//合并后的文件名
        .pipe(gulp.dest('dist/js'));
});

```
##  gulp-ruby-sass与ruby gem配置 
https://github.com/sindresorhus/gulp-ruby-sass
http://www.zuojj.com/archives/550.html?utm_source=tuicool&utm_medium=referral
首先得安装sass程序，不然会报错。而安装sass之前得安装ruby，这样才能通过gem安装sass。
Ruby安装:http://rubyinstaller.org/downloads/
然后通过gem install sass
检测是否正确:sass -v
不过貌似gem中央仓库被墙了。所以需要使用淘宝镜像。
**gem国内镜像配置**
gem淘宝源http://ruby.taobao.org/以及不再更新了。请看http://www.oschina.net/news/71749/taobao-gems-ruby-china
推荐使用https://gems.ruby-china.org/源
```

$ gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
$ gem sources -l
*** CURRENT SOURCES ***

https://gems.ruby-china.org
# 确保只有 gems.ruby-china.org
$ gem install sass

```
如果遇到 SSL 证书问题，你又无法解决，请直接用 http://gems.ruby-china.org 避免 SSL 的问题。
```

gulp = require('gulp');
sass = require('gulp-ruby-sass');
sourcemaps = require('gulp-sourcemaps');

gulp.task('sass', function(){
    sass('source/file.scss', {sourcemap: true})
        .on('error', sass.logError)
        // for inline sourcemaps
        .pipe(sourcemaps.write())
        // for file sourcemaps
        .pipe(sourcemaps.write('maps', {
            includeContent: false,
            sourceRoot: 'source'
        }))
        .pipe(gulp.dest('result'))
});

```

##  使用示例 
```

var gulp = require('gulp'),
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload'),
    del = require('del'),
    cssver=require('gulp-make-css-url-version'),
    sourcemaps=require('gulp-sourcemaps');
  gulp.task('testJs',function(){
  	return gulp.src(['common/**/*.js','!**/block.js'])
  			.pipe(sourcemaps.init())
  			.pipe(rename({suffix:'.min'}))
  			.pipe(uglify({
  				mangle:false
  			}))
  			.pipe(concat('all.js'))
  			.pipe(sourcemaps.write())
  			.pipe(gulp.dest('dest/js'));
  });
  gulp.task('testSass',function(){
  	return sass('common/*.scss', {})
  		.pipe(gulp.dest('dest'));
  });
  gulp.task('testCss',['testSass'],function(){
  	return gulp.src(['common/**/*.css'])
  			.pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
  			.pipe(gulp.dest('dist/prefix'))
    		.pipe(cssver())
  			.pipe(minifycss({
  				compatibility:'ie8'
  			}))
  			.pipe(rename({suffix:'.min'}))
  			.pipe(gulp.dest('dest/css'));
  });
  gulp.task('testHtml',function(){
  	return gulp.src(['common/index.html'])
  			.pipe(cssver())
  			.pipe(gulp.dest('dest/html'));
  });
  gulp.task('testImg',function(){
  	return gulp.src(['common/**/*.{png,jpg,ico,gif}'])
  			.pipe(cache(imagemin()))
  			.pipe(gulp.dest('dest/image'));  	
  });
  gulp.task('clean',function(callback){
  	return del(['dest/js','dest/css','dest/html','dest/image'],callback);

  });
  gulp.task('default',['clean'],function(){
  	gulp.start('testJs','testCss','testHtml');
  });

  gulp.task('watch',function(){
  	gulp.watch('common/*.scss',['testSass']);
  });

```