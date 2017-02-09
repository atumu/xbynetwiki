title: vuejs2 

#  Vue.js进阶 
##  自定义指令 
**自定义指令提供一种机制将数据的变化映射为 DOM 行为。**
可以注册一个全局自定义指令，也可以注册一个局部自定义指令。
**钩子函数**
定义对象可以提供几个钩子函数（**都是可选的**）：
  * bind：只调用一次，在指令第一次绑定到元素上时调用。
  * update： 在 bind 之后立即以初始值为参数第一次调用，之后每当绑定值变化时调用，参数为新值与旧值。
  * unbind：只调用一次，在指令从元素上解绑时调用。
```

Vue.directive('my-directive', {
  bind: function () {
    // 准备工作
    // 例如，添加事件处理器或只需要运行一次的高耗任务
  },
  update: function (newValue, oldValue) {
    // 值更新时的工作
    // 也会以初始值为参数调用一次
  },
  unbind: function () {
    // 清理工作
    // 例如，删除 bind() 添加的事件监听器
  }
})

```
在注册之后，便可以在 Vue.js 模板中这样用**（记着添加前缀 v-）**：
```

<div v-my-directive="someValue"></div>

```
指令实例属性
所有的钩子函数将被复制到实际的指令对象中，**钩子内 this 指向这个指令对象**。这个对象暴露了一些有用的属性：
  * el: 指令绑定的元素。
  * vm: 拥有该指令的上下文 ViewModel。
  * expression: 指令的表达式，不包括参数和过滤器。
  * arg: 指令的参数。
  * name: 指令的名字，不包含前缀。
  * modifiers: 一个对象，包含指令的修饰符。
  * descriptor: 一个对象，包含指令的解析结果。
```

<div id="demo" v-demo:hello.a.b="msg"></div>
Vue.directive('demo', {
  bind: function () {
    console.log('demo bound!')
  },
  update: function (value) {
    this.el.innerHTML =
      'name - '       + this.name + '<br>' +
      'expression - ' + this.expression + '<br>' +
      'argument - '   + this.arg + '<br>' +
      'modifiers - '  + JSON.stringify(this.modifiers) + '<br>' +
      'value - '      + value
  }
})
var demo = new Vue({
  el: '#demo',
  data: {
    msg: 'hello!'
  }
})

```
对象字面量
**如果指令需要多个值，可以传入一个 JavaScript 对象字面量。记住，指令可以使用任意合法的 JavaScript 表达式：**
```

<div v-demo="{ color: 'white', text: 'hello!' }"></div>
Vue.directive('demo', function (value) {
  console.log(value.color) // "white"
  console.log(value.text) // "hello!"
})        

```
###  params 
自定义指令可以接收一个** params 数组**，指定一个特性列表，Vue 编译器将自动提取绑定元素的这些特性。例如：
```

<div v-example a="hi"></div>
Vue.directive('example', {
  params: ['a'],
  bind: function () {
    console.log(this.params.a) // -> "hi"
  }
})

```
**此 API 也支持动态属性。this.params[key] 会自动保持更新。另外，可以指定一个回调，在值变化时调用：**
```

<div v-example v-bind:a="someValue"></div>
Vue.directive('example', {
  params: ['a'],
  paramWatchers: {
    a: function (val, oldVal) {
      console.log('a changed!')
    }
  }
})

```
类似于 props，指令参数的名字在 JavaScript 中使用 camelCase 风格，在 HTML 中对应使用 kebab-case 风格。

###  deep 
如果自定义指令用在一个对象上，当对象内部属性变化时要触发 update，则在指令定义对象中指定 deep: true。
```

<div v-my-directive="obj"></div>
Vue.directive('my-directive', {
  deep: true,
  update: function (obj) {
    // 在 `obj` 的嵌套属性变化时调用
  }
})

```

###  twoWay 
如果指令想向 Vue 实例写回数据，则在指令定义对象中指定 twoWay: true 。该选项允许在指令中使用 this.set(value):
```

Vue.directive('example', {
  twoWay: true,
  bind: function () {
    this.handler = function () {
      // 将数据写回 vm
      // 如果指令这样绑定 v-example="a.b.c"
      // 它将用给定值设置 `vm.a.b.c`
      this.set(this.el.value)
    }.bind(this)
    this.el.addEventListener('input', this.handler)
  },
  unbind: function () {
    this.el.removeEventListener('input', this.handler)
  }
})

```
###  priority 
可以给指令指定一个优先级。如果没有指定，普通指令默认是 1000。值越大，优先级越高。

##  自定义过滤器 
类似于自定义指令，可以用全局方法 Vue.filter() 注册一个自定义过滤器，它接收两个参数：过滤器 ID 和过滤器函数。
**过滤器函数以值为参数，返回转换后的值：**
```

Vue.filter('reverse', function (value) {
  return value.split(` ).reverse().join( `)
})
<!-- 'abc' => 'cba' -->
<span v-text="message | reverse"></span>
过滤器函数可以接收任意数量的参数：
Vue.filter('wrap', function (value, begin, end) {
  return begin + value + end
})
<!-- 'hello' => 'before hello after' -->
<span v-text="message | wrap 'before' 'after'"></span>

```
**双向过滤器**
目前我们使用过滤器都是在把来自模型的值显示在视图之前转换它。不过也可以定义一个过滤器，在把来自视图（<input> 元素）的值写回模型之前转化它：
```

Vue.filter('currencyDisplay', {
  // model -> view
  // 在更新 `<input>` 元素之前格式化值
  read: function(val) {
    return '$'+val.toFixed(2)
  },
  // view -> model
  // 在写回数据之前格式化值
  write: function(val, oldVal) {
    var number = +val.replace(/[^\d.]/g, '')
    return isNaN(number) ? 0 : parseFloat(number.toFixed(2))
  }
})

```
###  动态参数 
如果过滤器参数没有用引号包起来，则它会在当前 vm 作用域内动态计算。另外，**过滤器函数的 this 始终指向调用它的 vm。**例如：
```

<input v-model="userInput">
<span>![](/data/dokuwikimsg | concat userInput)</span>
Vue.filter('concat', function (value, input) {
  // `input` === `this.userInput`
  return value + input
})

```
**对于更复杂的逻辑——需要多于一个语句，这时需要将它放到计算属性或自定义过滤器中。**

##  混合Mixin与代码复用 
混合以一种灵活的方式为组件提供分布复用功能。混合对象可以包含任意的组件选项。当组件使用了混合对象时，混合对象的所有选项将被“混入”组件自己的选项中。
```

// 定义一个混合对象
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 定义一个组件，使用这个混合对象
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // -> "hello from mixin!"

```
**选项合并**
当混合对象与组件包含同名选项时，这些选项将以适当的策略合并。例如，同名钩子函数被并入一个数组，因而都会被调用。另外，混合的钩子将在组件自己的钩子之前调用。
值为对象的选项，如 methods, components 和 directives 将合并到同一个对象内。如果键冲突则组件的选项优先。
###  全局混合 
也可以全局注册混合。小心使用！一旦全局注册混合，它会影响所有之后创建的 Vue 实例。如果使用恰当，可以为自定义选项注入处理逻辑：
```

// 为 `myOption` 自定义选项注入一个处理器
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// -> "hello!"

```
慎用全局混合，因为它影响到每个创建的 Vue 实例，包括第三方组件。在大多数情况下，它应当只用于自定义选项

##  插件 
开发插件：略。参考文档http://cn.vuejs.org/guide/plugins.html
###  使用插件 
通过 Vue.use() 全局方法使用插件：
```

// 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin)
也可以传入一个选项对象：
Vue.use(MyPlugin, { someOption: true })

```
##  构建大型应用 
http://cn.vuejs.org/guide/application.html
###  已有插件 & 工具 
**[vue-router](https://github.com/vuejs/vue-router)**：Vue.js 官方路由。与 Vue.js 内核深度整合，让构建单页应用易如反掌。
[vue-resource](https://github.com/vuejs/vue-resource)：通过 XMLHttpRequest 或 JSONP 发起请求并处理响应。
vue-async-data：异步加载数据插件。
vue-validator：表单验证插件。
vue-devtools：Chrome 开发者工具扩展，用于调试 Vue.js 应用。
vue-touch：使用 Hammer.js 添加触摸手势指令。
vue-element：使用 Vue.js 注册自定义元素。
vue-animated-list： 方便的为 v-for 渲染的列表添加动画。

Vue 的主要组件
  * Vue - 核心库。
  * [Vuex](https://github.com/vuejs/vuex) - 类 Flux 的应用架构。
  * Vue-router - 单页面应用路由。
  * Vue-resource - 网络请求插件，使用 XMLHttpRequests 或 JSONP 处理响应。