location:web/tool/gulp组织小型项目小记
author:xbynet
createAt:2016-12-12 03:24:33
modifyAt:2016-12-13 01:41:51
title:gulp组织小型项目小记

目前正在开发一个python markdown wiki系统，对于前端模块化与打包这块出现了一些选择。
1、采用webpack模块化及打包
由于项目比较小，稍微了解后，觉得没必要采用webpack。杀鸡焉用牛刀？
2、采用requirejs模块化，gulp打包
还是由于项目比较小，甚至不需要进行模块化，所以放弃采用requirejs,只是采用gulp进行打包。
3、css预编译框架,目前比较流行的有sass,less:目前没有采用，下一步尝试一下。
4、js组件化，目前比较流行的如Angular,VueJs等。项目较小,需要ajax交互更新页面的并不多，没有采用。部分功能用模板引擎的宏来实现。

当然采用gulp打包主要是为了解决以下几个问题：
1、js、css文件压缩与合并
2、js、css缓存与版本问题

关于js、css缓存与版本问题问题，目前为解决新版本发布，使得浏览器缓存发送新的请求而不是使用缓存的js，css文件。有两种方案：

 - app.js变为app-2qwee.js
 - app.js变为app.js?v=wqe2434er3

我比较喜欢的是后者方式。
gulp有个插件`gulp-version-number`可以解决这个问题，但是由于我采用模板引擎`jinja2与flask`框架开发，所以html中是这样的

    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='app.css') }}">

经过gulp-version-number编译后变成了

    <link rel="stylesheet" type="text/css" href="{{ url_for('?v=16124df4fdca2e5244636c2bca625276static', filename='app.css') }}">
    
而我期望的是

    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='app.css') }}?v=16124df4fdca2e5244636c2bca625276">
好奇的你肯定会问，为什么期望的不是这样呢？
    
    url_for('static', filename='app.css?v=16124df4fdca2e5244636c2bca625276') }}"
    
经实验，经过url_for输出结果中?变成了%3F，经过了urlencode，所以浏览器无法识别它为查询参数。

没办法，只能改改这个插件的源码：具体代码在插件代码的index.js处：大概在207-284行部分：
主要改动就是替换sts[i].match正则，加入_RULE长度判断。

```
   var appendto = {
        'css': function (content, k, v) {
            var sts = content.match(/<link[^>]*rel=['"]?stylesheet['"]?[^>]*>/g);
			
            if (util.isArray(sts) && sts.length) {
                for (var i = 0, len = sts.length; i < len; i++) {
					
                    //var _RULE = sts[i].match(/href=['"]?([^>'"]*)['"]?/);
                    var _RULE=sts[i].match(/(\{\{\surl_for\('static',\s?filename=['"].*?['"]\s?\)\s\}\})/);
					if(!_RULE||_RULE.length<2){
						continue;
					}
					console.log(sts[i]);
					console.log(_RULE[1]);
					if (_RULE[1]) {
                        var _UrlPs = parseURL(_RULE[1]);
                        var _Query = queryToJson(_UrlPs.query);
                        var _Append = {};
                        if (!_Query.hasOwnProperty(k) || this['cover']) {
                            _Append[k] = v;
                        }
                        _UrlPs.query = jsonToQuery(util._extend(_Query, _Append));
                        content = content.replace(sts[i], sts[i].replace(_RULE[1], renderingURL(_UrlPs)));
                    }
                }
            }
            return content;
        },
        'js': function (content, k, v) {
            var sts = content.match(/<script[^>]*src=['"]?([^>'"]*)['"]?[^>]*>[^<]*<\/script>/g);
            if (util.isArray(sts) && sts.length) {
                for (var i = 0, len = sts.length; i < len; i++) {
                    var _RULE = sts[i].match(/(\{\{\surl_for\('static',\s?filename=['"].*?['"]\s?\)\s\}\})/);//.match(/src=['"]?([^>'"]*)['"]?/);
                    if(!_RULE||_RULE.length<2){
						continue;
					}
					console.log(sts[i]);
					console.log(_RULE[1]);
					
					if (_RULE[1]) {
                        var _UrlPs = parseURL(_RULE[1]);
                        var _Query = queryToJson(_UrlPs.query);
                        var _Append = {};
                        if (!_Query.hasOwnProperty(k) || this['cover']) {
                            _Append[k] = v;
                        }
                        _UrlPs.query = jsonToQuery(util._extend(_Query, _Append));
                        content = content.replace(sts[i], sts[i].replace(_RULE[1], renderingURL(_UrlPs)));
                    }
                }
            }
            return content;
        },
        'image': function (content, k, v) {
            var sts = content.match(/<img[^>]*>/g);
            if (util.isArray(sts) && sts.length) {
                for (var i = 0, len = sts.length; i < len; i++) {
                    var _RULE = sts[i].match(/(\{\{\surl_for\('static',\s?filename=['"].*?['"]\s?\)\s\}\})/);//.match(/src=['"]?([^>'"]*)['"]?/);
					if(!_RULE||_RULE.length<2){
						continue;
					}
                    console.log(sts[i]);
					console.log(_RULE[1]);
					if (_RULE[1]) {
                        var _UrlPs = parseURL(_RULE[1]);
                        var _Query = queryToJson(_UrlPs.query);
                        var _Append = {};
                        if (!_Query.hasOwnProperty(k) || this['cover']) {
                            _Append[k] = v;
                        }
                        _UrlPs.query = jsonToQuery(util._extend(_Query, _Append));
                        content = content.replace(sts[i], sts[i].replace(_RULE[1], renderingURL(_UrlPs)));
                    }
                }
            }
            return content;
        }
    };
```

然后，项目主要配置文件如下：不多说，具体看配置
package.json

```
{
  "name": "mdwiki",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "xbynet",
  "license": "ISC",
  "devDependencies": {
    "del": "^2.2.2",
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^3.1.1",
    "gulp-clean": "^0.3.2",
    "gulp-concat": "^2.6.1",
    "gulp-flatten": "^0.3.1",
    "gulp-minify-css": "^1.2.4",
    "gulp-rev": "^7.1.2",
    "gulp-sourcemaps": "^1.9.1",
    "gulp-uglify": "^2.0.0",
    "gulp-version-number": "^0.1.5",
    "pump": "^1.0.1",
    "run-sequence": "^1.2.2"
  }
}

```

gulpfile.js

```
const gulp = require('gulp');
const uglify = require('gulp-uglify');
const pump = require('pump');
const version = require('gulp-version-number');
const autoprefixer = require('gulp-autoprefixer');
const sourcemaps=require('gulp-sourcemaps');
const runSequence = require('run-sequence');
const del = require('del');
const cssmin = require('gulp-minify-css');
//const imagemin = require('gulp-imagemin');
const clean = require('gulp-clean');
//const flatten = require('gulp-flatten');
const DEST_DIR='build';
const SRC_DIR='mdwiki-master/app';
// Environment setup.
var env = {
    production: false
};

// Environment task.
gulp.task("set-production", function(){
    env.production = true;
});

const versionConfig = {
  'value': '%MDS%',
  'append': {
    'key': 'v',
    'to': ['css', 'js','image'],
  },
};

gulp.task('html',()=>{
	return gulp.src(SRC_DIR+'/**/*.html')
    .pipe(version(versionConfig))
    .pipe(gulp.dest(DEST_DIR));
});
gulp.task('jsmin', function (cb) {
	if(env.production){
		return gulp.src([SRC_DIR+'/**/*.js'])
        .pipe(uglify({mangle: {except: ['require' ,'exports' ,'module' ,'$']}})) //排除混淆关键字
        .pipe(gulp.dest(DEST_DIR));
	}
	else{
		return gulp.src([SRC_DIR+'/**/*.js'])
		.pipe(sourcemaps.init())
        .pipe(uglify({mangle: {except: ['require' ,'exports' ,'module' ,'$']}})) //排除混淆关键字
		.pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(DEST_DIR));
	}
	
});
gulp.task('cssmin', function () {
	if(env.production){
		return gulp.src(SRC_DIR+'/**/*.css')
		//.pipe(concat('main.css'))
        .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
        .pipe(cssmin())
        .pipe(gulp.dest(DEST_DIR));
	}
	else{
		return gulp.src(SRC_DIR+'/**/*.css')
		//.pipe(concat('main.css'))
        .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
		.pipe(sourcemaps.init())
		
        .pipe(cssmin())
		.pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(DEST_DIR));
	}
    
});
gulp.task('rawtoone', () =>{
    return gulp.src([SRC_DIR+'/**/*.{png,jpg,jpeg,gif,bmp,ico}',SRC_DIR+'/**/*.{swf,eot,svg,ttf,woff}'])
        .pipe(flatten())
		//.pipe(imagemin())
        .pipe(gulp.dest(DEST_DIR));
	}
);
gulp.task('copy', () =>{
    return gulp.src([SRC_DIR+'/**/*.{png,jpg,jpeg,gif,bmp,ico}',SRC_DIR+'/**/*.{swf,eot,svg,ttf,woff}'])
        .pipe(gulp.dest(DEST_DIR));
	}
);
// Clean
gulp.task('clean', function(cb) {
   return  del(['./build'],cb);
      // return gulp.src('build', {read: false,force: true})
      //  .pipe(clean());
});
// Default task
gulp.task('dev',function(callback) {
    runSequence('clean',['html', 'jsmin','cssmin','copy'],callback);
});

gulp.task('product',function(callback) {
    runSequence('set-production','clean',['html', 'jsmin','cssmin','copy'],callback);
});
gulp.task('watch', function() {
  gulp.watch(SRC_DIR+'/**/*.html',['html']);
  gulp.watch(SRC_DIR+'/**/*.js', ['jsmin']);
  gulp.watch(SRC_DIR+'/**/*.css', ['cssmin']);
  gulp.watch([SRC_DIR+'/**/*.{png,jpg,jpeg,gif,bmp,ico}',SRC_DIR+'/**/*.{swf,eot,svg,ttf,woff}'],['copy']);
  // Create LiveReload server
  //livereload.listen();
  // Watch any files in dist/, reload on change
  //gulp.watch(['dist/**']).on('change', livereload.changed);
});
```