modifyAt:2016-12-13 01:06:02
location:web/tool/gulp排除已压缩文件思路
title:gulp排除已压缩文件思路
author:xbynet
createAt:2016-12-13 01:06:02

# gulp默认排除语法的弊端

有个时候我们需要时用gulp排除已经压缩过的js，css等。如果以压缩文件是以".min.js"之类命名规范的还好，如果不是呢？而且还有其他一些场景也会有需要。
gulp默认支持glob格式，排除语法为!
举几个例子：
[排除目录][1]

    gulp.src([
        baseDir + '/**',                              // Include all
        '!' + baseDir + '/excl1{,/**}',               // Exclude excl1 dir
        '!' + baseDir + '/excl2/**/!(.gitignore)',    // Exclude excl2 dir, except .gitignore
    ], { dot: true });

[排除文件：][2]

    gulp.src(['css/**/!(ignore.css)*.css'])
    gulp.src([
          '!css/ignnore.css',
          'css/*.css'
        ])

这确实能够排除一些文件。但是还不算强大，不能满足一些复杂的场景。
而且还存在这样一个问题：当我有一个jquery.min.js，在压缩js时，我不想对它进行压缩，但同时我又想将其输出到dist目录下。单纯采用上面这种方式需要"曲线救国"，场景如下:

```
gulp.task('jsmin', function (cb) {
        return gulp.src([SRC_DIR+'/**/*.js',,'!'+SRC_DIR+'/**/*.min.js'])
        .pipe(sourcemaps.init())
        .pipe(uglify({mangle: {except: ['require' ,'exports' ,'module' ,'$']}})) //排除混淆关键字
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(DEST_DIR));

});
```
很遗憾地告诉你，*.min.js并没有被拷贝到DEST_DIR,当然曲线救国可以如下：

```
gulp.task('copy', () =>{
    return gulp.src([SRC_DIR+'/**/*.min.js'])
        .pipe(gulp.dest(DEST_DIR));
    }
);
```
但是，这不是很优雅。而且上面我们也说了只是采用glob语法对文件进行排除不适用于复杂的场景。

# Vinyl文件系统
详细说明，请参考[探究Gulp的Stream][3]
现在这里贴出，后面会说明其目的。
虽然Gulp使用的是Stream，但却不是普通的Node Stream，实际上，Gulp（以及Gulp插件）用的应该叫做Vinyl File Object Stream。
这里的Vinyl，是一种虚拟文件格式。**Vinyl主要用两个属性来描述文件，它们分别是路径（path）及内容（contents）**。具体来说，Vinyl并不神秘，它仍然是JavaScript Object。Vinyl官方给了这样的示例：

    var File = require('vinyl');
    var coffeeFile = new File({
      cwd: "/",
      base: "/test/",
      path: "/test/file.coffee",
      contents: new Buffer("test = 123")
    });

**从这段代码可以看出，Vinyl是Object，path和contents也正是这个Object的属性。**

# 解决方案描述：
可以采用gulp插件来实现更为强大的排除。主要有以下几个插件：
[gulp-ignore][4]，支持通过boolean,[stat][5]对象，函数来判断是否排除与包含。判断函数接受一个Vinyl文件对象，返回一个boolean值。
[gulp-filter][6]，支持glob模式、函数过滤，以及过滤后恢复
[gulp-if][7]，支持条件判断(支持函数条件)来控制相关流程。
下面具体说明如何使用

# gulp-ignore：
API
exclude(condition [, minimatchOptions])
Exclude files whose `file.path` matches, include everything else
include(condition [, minimatchOptions])
Include files whose `file.path` matches, exclude everything else
参数说明：
condition：类型可以为 boolean or stat object or function
当为函数是接受一个vinyl file返回boolean值

下面贴出官方例子：

```
var gulpIgnore = require('gulp-ignore');
var uglify = require('gulp-uglify');
var jshint = require('gulp-jshint');

var condition = './gulpfile.js';

gulp.task('task', function() {
  gulp.src('./**/*.js')
    .pipe(jshint())
    .pipe(gulpIgnore.exclude(condition))
    .pipe(uglify())
    .pipe(gulp.dest('./dist/'));
});
```

```
var gulpIgnore = require('gulp-ignore');
var uglify = require('gulp-uglify');
var jshint = require('gulp-jshint');

var condition = './public/**.js';

gulp.task('task', function() {
  gulp.src('./**/*.js')
    .pipe(jshint())
    .pipe(gulpIgnore.include(condition))
    .pipe(uglify())
    .pipe(gulp.dest('./dist/'));
});
```
实现：
```
var condition = function(f){
    if(f.path.endswith('.min.js')){
        return true;
    }
    return false
};

gulp.task('jsmin', function (cb) {
        return gulp.src([SRC_DIR+'/**/*.js'])
        .pipe(gulpIgnore.exclude(condition))
        .pipe(sourcemaps.init())
        .pipe(uglify({mangle: {except: ['require' ,'exports' ,'module' ,'$']}})) //排除混淆关键字
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(DEST_DIR));

});

```
优点：很明显，你可以在条件函数中做你想做的任何事情，包括使用正则表达式等。
缺点：和默认情况一样，不支持被排除文件的拷贝

# gulp-if
和gulp-ignore作者是同一个人
API：
gulpif(condition, stream [, elseStream, [, minimatchOptions]])
**gulp-if will pipe data to stream whenever condition is truthy.**
**If condition is falsey and elseStream is passed, data will pipe to elseStream**
这就类似于三目运算符,功能用伪代码表示就是if condition then stream(data) else elseStream(data)
After data is piped to stream or elseStream or neither, data is piped down-stream.

参数说明：
condition 与gulp-ignore一致。
Type: boolean or stat object or function that takes in a vinyl file and returns a boolean or RegularExpression that works on the file.path

stream
Stream for gulp-if to pipe data into when condition is truthy.

elseStream
Optional, Stream for gulp-if to pipe data into when condition is falsey.

minimatchOptions
Optional, if it's a glob condition, these options are passed to minimatch.


官方例子：
```
var gulpif = require('gulp-if');
var uglify = require('gulp-uglify');

var condition = true; // TODO: add business logic

gulp.task('task', function() {
  gulp.src('./src/*.js')
    .pipe(gulpif(condition, uglify()))
    .pipe(gulp.dest('./dist/'));
});
```
Only uglify the content if the condition is true, but send all the files to the dist folder
仅压缩符合条件的文件，丹斯所有文件(包括不符合条件的)都会被发送到dist目录。

实现：
```
var condition = function(f){
    if(f.path.endswith('.min.js')){
        return false;
    }
    return true;
};

gulp.task('jsmin', function (cb) {
        return gulp.src([SRC_DIR+'/**/*.js'])
        .pipe(sourcemaps.init())
        .pipe(gulpif(condition, uglify({mangle: {except: ['require' ,'exports' ,'module' ,'$']}}))) //排除混淆关键字
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(DEST_DIR));

});

```

# gulp-filter
API
filter(pattern, [options])
Returns a transform stream with a `.restore` object.
参数说明：
pattern
Type: **string, array, function**
Accepts a string/array with globbing patterns which are run through multimatch.
如果是函数，则应该接受一个 vinyl file object作为第一个参数，return true/false whether to include the file:

    filter(file => /unicorns/.test(file.path));

选项options
Type: object
Accepts minimatch options.
**Note: Set dot: true if you need to match files prefixed with a dot (eg. .gitignore).**

**options.restore**
Type: boolean
Default: false
Restore filtered files.

**options.passthrough**
Type: boolean
Default: true

When set to true filtered files are restored with a PassThrough stream, otherwise, when set to false, filtered files are restored as a Readable stream.

官方例子：

```
const gulp = require('gulp');
const uglify = require('gulp-uglify');
const filter = require('gulp-filter');

gulp.task('default', () => {
    // create filter instance inside task function
    const f = filter(['**', '!*src/vendor'], {restore: true});

    return gulp.src('src/**/*.js')
        // filter a subset of the files
        .pipe(f)
        // run them through a plugin
        .pipe(uglify())
        // bring back the previously filtered out files (optional)
        .pipe(f.restore)
        .pipe(gulp.dest('dist'));
});
gulp.task('defaultmulti', () => {
    const jsFilter = filter('**/*.js', {restore: true});
    const lessFilter = filter('**/*.less', {restore: true});

    return gulp.src('assets/**')
        .pipe(jsFilter)
        .pipe(concat('bundle.js'))
        .pipe(jsFilter.restore)
        .pipe(lessFilter)
        .pipe(less())
        .pipe(lessFilter.restore)
        .pipe(gulp.dest('out/'));
});
```

实战：

    var condition = function(f){
            if(f.path.endswith('.min.js')){
                return false;
            }
            return true;
        };
    const jsFilter = filter(condition, {restore: true});
       
        gulp.task('jsmin', function (cb) {
                return gulp.src([SRC_DIR+'/**/*.js'])
                .pipe(jsFilter)
                .pipe(sourcemaps.init())
                .pipe(uglify({mangle: {except: ['require' ,'exports' ,'module' ,'$']}})) //排除混淆关键字
                .pipe(sourcemaps.write('.'))
                .pipe(jsFilter.restore)
                .pipe(gulp.dest(DEST_DIR));
        
        });
部分文件被压缩，所有文件都会被发送到DEST_DIR

# 小结
如何选择？
gulp-if:可以完美解决问题，且足够优雅。
gulp-filter：可以完美解决问题，但是相比gulp-if略显麻烦。更复杂的场景下用gulp-filter比gulp-if方便。
gulp-ignore:更为强大的过滤，在仅仅只是过滤而不拷贝的情况下首推。


  [1]: http://stackoverflow.com/questions/26485612/glob-minimatch-how-to-gulp-src-everything-then-exclude-folder-but-keep-one
  [2]: http://stackoverflow.com/questions/26708110/how-to-tell-gulp-to-skip-or-ignore-some-files-in-gulp-src
  [3]: https://segmentfault.com/a/1190000003770541
  [4]: https://github.com/robrich/gulp-ignore
  [5]: https://nodejs.org/api/fs.html#fs_class_fs_stats
  [6]: https://github.com/sindresorhus/gulp-filter
  [7]: https://github.com/robrich/gulp-if