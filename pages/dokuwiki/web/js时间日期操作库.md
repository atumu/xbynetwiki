title: js时间日期操作库 

#  js时间日期操作库moment.js 
http://momentjs.com/
Format Dates
```

moment().format('MMMM Do YYYY, h:mm:ss a'); // 五月 5日 2016, 5:57:01 下午
moment().format('dddd');                    // 星期四
moment().format("MMM Do YY");               // 5月 5日 16
moment().format('YYYY [escaped] YYYY');     // 2016 escaped 2016
moment().format();                          // 2016-05-05T17:57:01+08:00

```
Relative Time
```

moment("20111031", "YYYYMMDD").fromNow(); // 5 年前
moment("20120620", "YYYYMMDD").fromNow(); // 4 年前
moment().startOf('day').fromNow();        // 18 小时前
moment().endOf('day').fromNow();          // 6 小时内
moment().startOf('hour').fromNow();       // 1 小时前
Calendar Time
moment().subtract(10, 'days').calendar(); // 2016年4月25日
moment().subtract(6, 'days').calendar();  // 上周五下午5点57
moment().subtract(3, 'days').calendar();  // 本周一下午5点57
moment().subtract(1, 'days').calendar();  // 昨天下午5点57分
moment().calendar();                      // 今天下午5点57分
moment().add(1, 'days').calendar();       // 明天下午5点57分
moment().add(3, 'days').calendar();       // 本周日下午5点57
moment().add(10, 'days').calendar();      // 2016年5月15日

```
Multiple Locale Support
```

moment.locale();         // zh-cn
moment().format('LT');   // 下午5点57分
moment().format('LTS');  // 下午5点57分1秒
moment().format('L');    // 2016-05-05
moment().format('l');    // 2016-05-05
moment().format('LL');   // 2016年5月5日
moment().format('ll');   // 2016年5月5日
moment().format('LLL');  // 2016年5月5日下午5点57分
moment().format('lll');  // 2016年5月5日下午5点57分
moment().format('LLLL'); // 2016年5月5日星期四下午5点57分
moment().format('llll'); // 2016年5月5日星期四下午5点57分

```