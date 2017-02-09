modifyAt:2016-12-18 13:34:26
title:Flask-Moment本地化时间日期 
createAt:2016-12-18 13:34:26
location:dokuwiki/python/flask-moment
author:xbynet


github:https://github.com/miguelgrinberg/Flask-Moment
Formatting of dates and times in Flask templates using moment.js.
当前版本:0.5.1
pip install flask-moment

Moment.js 是一个简单易用的轻量级JavaScript日期处理类库，提供了日期格式化、日期解析等功能。它支持在浏览器和NodeJS两种环境中运行。此类库能够 将给定的任意日期转换成多种不同的格式，具有强大的日期计算功能，同时也内置了能显示多样的日期形式的函数。另外，它也支持多种语言.
**Flask-Moment是一个集成moment.js到Jinja2模板的Flask扩展。**

```

from flask.ext.moment import Moment
moment = Moment(app)

或者使用init_app()

moment = Moment()
    def create_app(config):
        app = Flask(__name__)
        app.config.from_object(config)
        # initialize moment on the app within create_app()
        moment.init_app(app)

app = create_app(prod_config)

```

Flask-Moment依赖moment.js和jquery.js。需要直接包含在HTML文档
在base.html模版中的head标签中导入moment.js和jquery.js
如果使用了bootstrap,可以不用导入jquery.js,因为bootstrap中包含了jquery.js
```

<html>
    <head>

        {{ moment.include_jquery() }}
    <script type="text/javascript" src="{{ url_for('static', filename='lib/moment-with-locales.min.js') }} "></script>
    {{ moment.include_moment(local_js="") }}{#local_js=""fa防止为None时它去加载国外cdn#}
    {{ moment.lang("zh-CN") }}
　　　　 
</head> <body> ... </body> </html>

```
为了使用flask-moment需要传入一个时间变量渲染到模版中,如:
```

from flask import render_template
from datetime import date
@main.route('/')
def index ():
    return render_template('index.html', time = date(1994,8,29))

```
在模版中渲染,如:
```

<p>现在时间时: ![](/data/dokuwiki moment().format('YYYY年M月D日, h/mm/ss a') ).</p>
<p>已经过去了: ![](/data/dokuwiki moment().fromTime(time) ).</p>
<p>![](/data/dokuwiki moment().calendar() ).</p>

```
结果
现在时间时: 2015年4月22日, 10:06:33 上午.
已经过去了: 21年内.
今天上午10点06.
在moment()中如果不传入python的时间变量,则默认将utc时间转换成本地时间作为显示,传入local=True参数可以关闭转换.
三.常用格式化参数
http://momentjs.com/docs/#/parsing/string/
Year, month, and day tokens
YYYY	2014	年份
YY	14	2个字符表示的年份
Q	1..4	季度
M MM	4..04	月份
MMM MMMM	4月..四月	根据moment.locale()中的设置显示月份
D DD	1..31	一月中的第几天
Do	1日..31日	一月中的第几天
DDD DDDD	1..365	一年中的第几天
X	1410715640.579	时间戳
x	1410715640579	时间戳

Hour, minute, second, millisecond, and offset tokens
H HH	0..23	24 hour time
h hh	1..12	12 hour time used with a A.
a A	am pm	Post or ante meridiem (Note the one character a p are also considered valid)
m mm	0..59	Minutes
s ss	0..59	Seconds
S SS SSS	0..999	Fractional seconds
Z ZZ	+12:00	Offset from UTC as +-HH:mm, +-HHmm, or Z

Week year, week, and weekday tokens
gggg	2014	Locale 4 digit week year
gg	14	Locale 2 digit week year
w ww	1..53	Locale week of year
e	0..6	Locale day of week
ddd dddd	Mon...Sunday	Day name in locale set by moment.locale()
GGGG	2014	ISO 4 digit week year
GG	14	ISO 2 digit week year
W WW	1..53	ISO week of year
E	1..7	ISO day of week