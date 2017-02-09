title: vuejs3 

#  Vue.js修改定界符 
参考：
http://vuejs.org.cn/api/#delimiters
https://segmentfault.com/q/1010000006702019
https://segmentfault.com/q/1010000007035319

在所有的项目中都是这样定义的，写在VUE代码的前面即可，这样代码迁移也不会出问题
```

//模板字符串
Vue.config.delimiters = ['${', '}']
// 修改文本插值的定界符![](/data/dokuwiki )。

Vue.config.unsafeDelimiters = ['{!!', '!!}']
// 修改原生 HTML 插值的定界符{![](/data/dokuwiki )}。

```