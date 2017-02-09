title: vuejs1 

#  Vue.js基础 
Vue.js是一个构建数据驱动的 web 界面的库。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的**视图组件**。它只聚焦于视图层。
官网：http://vuejs.org/
中文：http://cn.vuejs.org/
最新版本2.0.3，本文版本:1.0.26

1.0版本API：http://cn.vuejs.org/api/

特性：
简洁：HTML 模板 + JSON 数据，再创建一个 Vue 实例，就这么简单。
数据驱动：自动追踪依赖的模板表达式和计算属性。
组件化：用解耦、可复用的组件来构造界面。
轻量：~24kb min+gzip，无依赖。
快速：精确有效的异步批量 DOM 更新。
模块友好：AMD、CommonJS支持等

安装：http://cn.vuejs.org/guide/installation.html

Hello World
```

<div id="app">
  ![](/data/dokuwiki message )
</div>

new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  }
})

Hello Vue.js!

```

**双向绑定**
```

<div id="app">
  <p>![](/data/dokuwiki message )</p>
  <input v-model="message">
</div>
  
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  }
})

```

**渲染列表**
```

<div id="app">
  <ul>
    <li v-for="todo in todos">
      ![](/data/dokuwiki todo.text )
    </li>
  </ul>
</div>
new Vue({
  el: '#app',
  data: {
    todos: [
      { text: 'Learn JavaScript' },
      { text: 'Learn Vue.js' },
      { text: 'Build Something Awesome' }
    ]
  }
})

```

**处理用户输入**
```

<div id="app">
  <p>![](/data/dokuwiki message )</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split(` ).reverse().join( `)
    }
  }
})

```

**综合**
```

<div id="app">
  <input v-model="newTodo" v-on:keyup.enter="addTodo">
  <ul>
    <li v-for="todo in todos">
      <span>![](/data/dokuwiki todo.text )</span>
      <button v-on:click="removeTodo($index)">X</button>
    </li>
  </ul>
</div>
  
new Vue({
  el: '#app',
  data: {
    newTodo: '',
    todos: [
      { text: 'Add some todos' }
    ]
  },
  methods: {
    addTodo: function () {
      var text = this.newTodo.trim()
      if (text) {
        this.todos.push({ text: text })
        this.newTodo = ''
      }
    },
    removeTodo: function (index) {
      this.todos.splice(index, 1)
    }
  }
})

Add some todos  X

```
v-if 特性被称为指令。指令带有前缀 v-，用于控制元素是否添加/移除。这一点与v-show是由区别的。
v-for 指令用于显示数组元素，v-bind 指令用于绑定 HTML 特性

##  Vue对象 
每个 Vue.js 应用的起步都是通过构造函数 **Vue** 创建一个 Vue 的根实例：
```

var vm = new Vue({
  // 选项
})

```
` 一个 Vue 实例其实正是一个 MVVM 模式中所描述的 ViewModel - 因此在文档中经常会使用 vm 这个变量名。 `
可以扩展 Vue 构造器，从而用预定义选项创建可复用的组件构造器
**属性与方法**
每个 Vue 实例都会**代理**其 data 对象里所有的属性：
```

var data = { a: 1 }
var vm = new Vue({
  data: data
})

vm.a === data.a // -> true

// 设置属性也会影响到原始数据
vm.a = 2
data.a // -> 2

// ... 反之亦然
data.a = 3
vm.a // -> 3

```
**注意只有这些被代理的属性是响应的。如果在实例创建之后添加新的属性到实例上，它不会触发视图更新。**
除了这些数据属性，**Vue 实例暴露了一些有用的实例属性与方法。这些属性与方法都有前缀 $，以便与代理的数据属性区分**。例如：
```

vm.$data === data // -> true
vm.$el === document.getElementById('example') // -> true
// $watch 是一个实例方法
vm.$watch('a', function (newVal, oldVal) {
  // 这个回调将在 `vm.a`  改变后调用
})

```

**生命周期**
Vue 实例在创建时有一系列初始化步骤——例如，它需要建立数据观察，编译模板，创建必要的数据绑定。
在此过程中，它也将调用一些生命周期钩子，给自定义逻辑提供运行机会。
例如 created 钩子在实例创建后调用， compiled、 ready 、destroyed。**钩子的 this 指向调用它的 Vue 实例。**
![](/data/dokuwiki/web/pasted/20161027-232315.png?500x1300)
##  数据绑定语法 
Vue.js 的模板是基于 DOM 实现的。这意味着所有的 Vue.js 模板都是可解析的有效的 HTML
插值
&lt;nowiki&gt;
数据绑定最基础的形式是文本插值&lt;span&gt;Message: ![](/data/dokuwiki msg )&lt;/span&gt;
你也可以只处理单次插值，今后的数据变化就不会再引起插值更新了：&lt;span&gt;This will never change: ![](/data/dokuwiki* msg )&lt;/span&gt;
双 Mustache 标签将数据解析为纯文本而不是 HTML。为了输出真的 HTML 字符串，需要用三 Mustache 标签：
&lt;div&gt;![](/data/dokuwiki{ raw_html )}&lt;/div&gt; 
内容以 HTML 字符串插入——数据绑定将被忽略。

Mustache 标签也可以用在 HTML 特性 (Attributes) 内：
&lt;div id="item-![](/data/dokuwiki id )"&gt;&lt;/div&gt;
&lt;/nowiki&gt;
放在 Mustache 标签内的文本称为**绑定表达式**。
在 Vue.js 中，一段绑定表达式由一个简单的 JavaScript 表达式和可选的一个或多个过滤器构成。
Vue.js 在数据绑定内支持全功能的 JavaScript 表达式:如下：
&lt;nowiki&gt;
![](/data/dokuwiki number)
![](/data/dokuwiki number + 1 )
![](/data/dokuwiki ok ? 'YES' / 'NO' )
![](/data/dokuwiki message.split(` ).reverse().join( `) )
&lt;/nowiki&gt;

###  过滤器
&lt;nowiki&gt;
![](/data/dokuwiki message | capitalize )
过滤器可以串联：
![](/data/dokuwiki message | filterA | filterB )
过滤器也可以接受参数：
![](/data/dokuwiki message | filterA 'arg1' arg2 )
&lt;/nowiki&gt;
内置过滤器：
  * capitalize
  * uppercase
  * lowercase
  * currency
  * pluralize
  * **json**
  * debounce
  * limitBy
  * filterBy
  * orderBy

###  指令 
指令 (Directives) 是特殊的带有前缀 v- 的特性。指令的值限定为绑定表达式。
**指令的职责就是当其表达式的值改变时把某些特殊的行为应用到 DOM 上**。我们来回头看下“概述”里的例子：
<p v-if="greeting">Hello!</p>
这里 v-if 指令将根据表达式 greeting 值的真假**删除/插入 <p> 元素**。

**指令参数：**
```

<a v-bind:href="url"></a>
<a v-on:click="doSomething">

```
**指令修饰符**
例如 .literal 修饰符告诉指令将它的值解析为一个字面字符串而不是一个表达式：
```

<a v-bind:href.literal="/a/b/c"></a>

```

缩写
v-bind 缩写
```

<!-- 完整语法 -->
<button v-bind:disabled="someDynamicCondition">Button</button>
<!-- 缩写 -->
<button :disabled="someDynamicCondition">Button</button>

```
v-on 缩写
```

<!-- 完整语法 -->
<a v-on:click="doSomething"></a>
<!-- 缩写 -->
<a @click="doSomething"></a>

```
##  计算属性 
Vue.js 将绑定表达式限制为一个表达式。**如果需要多于一个表达式的逻辑，应当使用计算属性。**
基础例子
```

<div id="example">
  a=![](/data/dokuwiki a ), b=![](/data/dokuwiki b )
</div>
var vm = new Vue({
  el: '#example',
  data: {
    a: 1
  },
  computed: {
    // 一个计算属性的 getter
    b: function () {
      // `this` 指向 vm 实例
      return this.a + 1
    }
  }
})

```
###  计算属性 vs. $watch 
Vue.js 提供了一个方法 **$watch**，它用于观察 Vue 实例上的数据变动。
当一些数据需要根据其它数据变化时， $watch 很诱人。
不过，**通常更好的办法是使用计算属性而不是一个命令式的 $watch 回调**。考虑下面例子：
```

<div id="demo">![](/data/dokuwikifullName)</div>
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  }
})
vm.$watch('firstName', function (val) {
  this.fullName = val + ' ' + this.lastName
})
vm.$watch('lastName', function (val) {
  this.fullName = this.firstName + ' ' + val
})

```
上面代码是命令式的重复的。跟计算属性对比：
```

var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})

```
###  计算 setter 
计算属性默认只是 getter，不过在需要时你也可以提供一个 setter：
```

// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...

```
现在在调用 vm.fullName = 'John Doe' 时，setter 会被调用，vm.firstName 和 vm.lastName 也会有相应更新。

##  Class 与 Style 绑定 
数据绑定一个常见需求是操作元素的 class 列表和它的内联样式。
**在 v-bind 用于 class 和 style 时，Vue.js 专门增强了它。表达式的结果类型除了字符串之外，还可以是对象或数组。**

###  绑定 HTML Class 
我们可以传给 **v-bind:class** 一个对象，以动态地切换 class。注意 v-bind:class 指令可以与普通的 class 特性共存：
对象语法
```

<div class="static" v-bind:class="{ 'class-a': isA, 'class-b': isB }"></div>
data: {
  isA: true,
  isB: false
}

```
你也可以直接绑定数据里的一个对象：
```

<div v-bind:class="classObject"></div>
data: {
  classObject: {
    'class-a': true,
    'class-b': false
  }
}

```

**数组语法**
我们可以把一个数组传给 v-bind:class，以应用一个 class 列表：
```

<div v-bind:class="[classA, classB]">
data: {
  classA: 'class-a',
  classB: 'class-b'
}

```
如果你也想根据条件切换列表中的 class，可以用三元表达式：
```

<div v-bind:class="[classA, isB ? classB : '']">

```
###  绑定内联样式 
对象语法
**v-bind:style** 的对象语法十分直观——看着非常像 CSS，其实它是一个 JavaScript 对象。
CSS 属性名可以用驼峰式（camelCase）或短横分隔命名（kebab-case）：
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
直接绑定到一个样式对象通常更好，让模板更清晰：
```

<div v-bind:style="styleObject"></div>
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }

```
数组语法
v-bind:style 的数组语法可以将多个样式对象应用到一个元素上：
```

<div v-bind:style="[styleObjectA, styleObjectB]">

```
  
**自动添加前缀**
当 v-bind:style 使用需要厂商前缀的 CSS 属性时，如 transform，Vue.js 会自动侦测并添加相应的前缀。

##  条件/列表渲染 
**v-if**
```

<h1 v-if="ok">Yes</h1>
或者
<h1 v-if="ok">Yes</h1>
<h1 v-else>No</h1>

```
**template v-if**
如果我们想切换多个元素呢？此时我们可以把一个** <template> 元素当做包装元素，**并在上面使用 v-if，最终的渲染结果不会包含它。
```

<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>

```

**v-show**
另一个根据条件展示元素的选项是 v-show 指令。用法大体上一样：
<h1 v-show="ok">Hello!</h1>
不同的是有** v-show 的元素会始终渲染并保持在 DOM 中。v-show 是简单的切换元素的 CSS 属性 display。**
注意 v-show 不支持 <template> 语法。

**v-else**
可以用 v-else 指令给 v-if 或 v-show 添加一个 “else 块”，v-else 元素必须立即跟在 v-if 或 v-show 元素的后面——否则它不能被识别

一般来说，v-if 有更高的切换消耗而 v-show 有更高的初始渲染消耗。因此，如果需要频繁切换 v-show 较好，如果在运行时条件不大可能改变 v-if 较好。


----
**v-for**
内部特殊变量 $index，$key
```

<ul id="example-2">
  <li v-for="item in items">
    ![](/data/dokuwiki parentMessage ) - ![](/data/dokuwiki $index ) - ![](/data/dokuwiki item.message )
  </li>
</ul>

<li v-for="value in object">
    ![](/data/dokuwiki $key ) : ![](/data/dokuwiki value )
</li>

```

也可以这么用
<div v-for="(index, item) in items">
<div v-for="(key, val) in object">
或者
<div v-for="item of items">

**template v-for**
```

<ul>
  <template v-for="item in items">
    <li>![](/data/dokuwiki item.msg )</li>
    <li class="divider"></li>
  </template>
</ul>

```

###  数组变动检测 
变异方法
**Vue.js 包装了被观察数组的变异方法，故它们能触发视图更新。**被包装的方法有：
  * push()
  * pop()
  * shift()
  * unshift()
  * splice()
  * sort()
  * reverse()
相比之下，也有非变异方法，如 filter(), concat() 和 slice()，不会修改原始数组而是返回一个新数组。
在使用非变异方法时，可以直接用新数组替换旧数组：
```

example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})

```
**track-by**
有时需要用全新对象（例如通过 API 调用创建的对象）替换数组。使用 track-by 特性给 Vue.js 一个提示，Vue.js 因而能尽可能地复用已有实例。
具体看文档。http://cn.vuejs.org/guide/list.html#template_v-for

**值域 v-for**
v-for 也可以接收一个整数，此时它将重复模板数次。
```

 <span v-for="n in 10">![](/data/dokuwiki n ) </span>

```
##  方法与事件处理器 
方法处理器
可以用 v-on 指令监听 DOM 事件：
```

<button v-on:click="greet">Greet</button>
var vm = new Vue({
  el: '#example',
  data: {
    name: 'Vue.js'
  },
  // 在 `methods` 对象中定义方法
  methods: {
    greet: function (event) {
      // 方法内 `this` 指向 vm
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      alert(event.target.tagName)
    }
  }
})

// 也可以在 JavaScript 代码中调用方法
vm.greet() // -> 'Hello Vue.js!'

```

**内联语句处理器**
除了直接绑定到一个方法，也可以用内联 JavaScript 语句：
<button v-on:click="say('hi')">Say Hi</button>
以用特殊变量 $event 把它传入方法：
<button v-on:click="say('hello!', $event)">Submit</button>

###  事件修饰符 
在事件处理器中经常需要调用 **event.preventDefault()** 或** event.stopPropagation()**。
Vue.js 为 v-on 提供两个 事件修饰符：**.prevent 与 .stop**。
```

<!-- 阻止单击事件冒泡 -->
<a v-on:click.stop="doThis"></a>
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>
<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat">
<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>
<!-- 添加事件侦听器时使用 capture 模式 -->
<div v-on:click.capture="doThis">...</div>
<!-- 只当事件在该元素本身（而不是子元素）触发时触发回调 -->
<div v-on:click.self="doThat">...</div>

```
###  按键修饰符 
在监听键盘事件时，我们经常需要检测 keyCode。
Vue.js 允许为 v-on 添加按键修饰符：
```

<!-- 只有在 keyCode 是 13 时调用 vm.submit() -->
<input v-on:keyup.13="submit">
记住所有的 keyCode 比较困难，Vue.js 为最常用的按键提供别名：
<!-- 同上 -->
<input v-on:keyup.enter="submit">
<!-- 缩写语法 -->
<input @keyup.enter="submit">

```
全部的按键别名：
  * enter
  * tab
  * delete
  * esc
  * space
  * up
  * down
  * left
  * right
  * 支持单字母按键别名。
1.0.17+： 可以自定义按键别名：
```

// 可以使用 @keyup.f1
Vue.directive('on').keyCodes.f1 = 112

```
##  表单控件绑定 
可以用 **v-model 指令**在表单控件元素上创建**双向数据绑定**。
```

<span>Message is: ![](/data/dokuwiki message )</span>
<input type="text" v-model="message" placeholder="edit me">
  
单个勾选框，逻辑值：true,false
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">![](/data/dokuwiki checked )</label>

多个勾选框，绑定到同一个数组：
<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>
<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>
<span>Checked names: ![](/data/dokuwiki checkedNames | json )</span> 
Checked names: ["Jack","John"]
  
多选（绑定到一个数组）：
<select v-model="selected" multiple>
  <option selected>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>Selected: ![](/data/dokuwiki selected | json )</span>

```
  
**绑定 value**
有时我们想绑定 value 到 Vue 实例的一个动态属性上，这时可以用 v-bind 实现，并且这个属性的值可以不是字符串。
```

Checkbox
<input type="checkbox" v-model="toggle" v-bind:true-value="a" v-bind:false-value="b">
// 当选中时
vm.toggle === vm.a
// 当没有选中时
vm.toggle === vm.b
  
Radio
<input type="radio" v-model="pick" v-bind:value="a">
// 当选中时
vm.pick === vm.a
  
Select Options
<select v-model="selected">
  <!-- 对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
// 当选中时
typeof vm.selected // -> 'object'
vm.selected.number // -> 123

```

###  debounce延迟同步 
debounce 设置一个最小的延时，**在每次敲击之后延时同步输入框的值与数据。如果每次更新都要进行高耗操作（例如在输入提示中 Ajax 请求），它较为有用。**
```

<input v-model="msg" debounce="500">

```
` 注意 debounce延迟“写入”底层数据。因此在使用 debounce 时应当用 vm.$watch() 响应数据的变化。 `
##  过渡特效 
建议与动画特效css库[animate.css](https://github.com/daneden/animate.css)一起使用
通过 Vue.js 的过渡系统，**可以在元素从 DOM 中插入或移除时自动应用过渡效果。**
你也可以提供相应的 JavaScript **钩子函数**在过渡过程中执行自定义的 DOM 操作。
```

<div v-if="show" transition="my-transition"></div>

```
**transition 特性可以与下面资源一起用：**
  * v-if
  * v-show
  * v-for （只在插入和删除时触发，使用 vue-animated-list 插件）
  * 动态组件
  * 在组件的根节点上，并且被 Vue 实例 DOM 方法（如 vm.$appendTo(el)）触发。

当插入或删除带有过渡的元素时，Vue 将：
  * 尝试以 ID "my-transition" 查找 JavaScript 过渡**钩子对象**
  * 自动嗅探目标元素是否有 CSS 过渡或动画，**并在合适时添加/删除 CSS 类名。**

**CSS 过渡,CSS 动画**
详情见文档：http://cn.vuejs.org/guide/transitions.html


###  自定义过渡类名 
这些自定义类名会覆盖默认的类名。**当需要和第三方的 CSS 动画库，比如 Animate.css 配合时会非常有用：**
```

<div v-show="ok" class="animated" transition="bounce">Watch me bounce</div>
Vue.transition('bounce', {
  enterClass: 'bounceInLeft',
  leaveClass: 'bounceOutRight'
})

```

**显式声明 CSS 过渡类型**
比如你可能希望让 Vue 来触发一个 CSS animation，同时该元素在鼠标悬浮时又有 CSS transition 效果。这样的情况下，你需要显式地声明你希望 Vue 处理的动画类型 (animation 或是 transition)：
```

Vue.transition('bounce', {
  // 该过渡效果将只侦听 `animationend` 事件
  type: 'animation'
})

```

**JavaScript 过渡**
也可以只使用 JavaScript 钩子，不用定义任何 CSS 规则。
当只使用 JavaScript 过渡时，enter 和 leave 钩子需要调用 done 回调，否则它们将被同步调用，过渡将立即结束。
在下例中我们使用 jQuery 注册一个自定义的 JavaScript 过渡：
```

Vue.transition('fade', {
  css: false,
  enter: function (el, done) {
    // 元素已被插入 DOM
    // 在动画结束后调用 done
    $(el)
      .css('opacity', 0)
      .animate({ opacity: 1 }, 1000, done)
  },
  enterCancelled: function (el) {
    $(el).stop()
  },
  leave: function (el, done) {
    // 与 enter 相同
    $(el).animate({ opacity: 0 }, 1000, done)
  },
  leaveCancelled: function (el) {
    $(el).stop()
  }
})

```
然后用 transition 特性中：
<p transition="fade"></p>

**渐近过渡**
transition 与 v-for 一起用时可以创建渐近过渡。给过渡元素添加一个特性 stagger, enter-stagger 或 leave-stagger：
```

<div v-for="item in list" transition="stagger" stagger="100"></div>

```
或者，提供一个钩子 stagger, enter-stagger 或 leave-stagger，以更好的控制：
```

Vue.transition('stagger', {
  stagger: function (index) {
    // 每个过渡项目增加 50ms 延时
    // 但是最大延时限制为 300ms
    return Math.min(300, index * 50)
  }
})

```

##  组件 
组件（Component）是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。
在有些情况下，组件也可以是原生 HTML 元素的形式，以 **is** 特性扩展。
组件在注册之后，便可以在父实例的模块中以自定义元素 <my-component> 的形式使用。
**要确保在初始化根实例之前注册了组件：**
```

<div id="example">
  <my-component></my-component>
</div>
// 定义
var MyComponent = Vue.extend({
  template: '<div>A custom component!</div>'
})

// 全局注册组件，tag 为 my-component
Vue.component('my-component', MyComponent)

// 创建根实例
new Vue({
  el: '#example'
})

```
注意组件的模板替换了自定义元素，自定义元素的作用只是作为一个挂载点。

**局部注册**
不需要全局注册每个组件。可以让组件只能用在其它组件内，用实例选项 components 注册：
```

var Child = Vue.extend({ /* ... */ })
var Parent = Vue.extend({
  template: '...',
  components: {
    // <my-component> 只能用在父组件模板内
    'my-component': Child
  }
})

```
这种封装也适用于其它资源，如指令、过滤器和过渡。

**注册语法糖**
为了让事件更简单，可以直接传入选项对象而不是构造器给 Vue.component() 和 component 选项。
```

// 在一个步骤中扩展与注册
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})

// 局部注册也可以这么做
var Parent = Vue.extend({
  components: {
    'my-component': {
      template: '<div>A custom component!</div>'
    }
  }
})

```
###  组件选项问题 
传入 Vue 构造器的多数选项也可以用在 Vue.extend() 中，不**过有两个特例： data 和 el。**
试想如果我们简单地把一个对象作为 data 选项传给 Vue.extend()：
这么做的问题是** MyComponent 所有的实例将共享同一个 data 对象！这基本不是我们想要的**，因此我们应当使用一个函数作为 data 选项，让这个函数返回一个新对象：
```

var MyComponent = Vue.extend({
  data: function () {
    return { a: 1 }
  }
})

```
**同理，el 选项用在 Vue.extend() 中时也须是一个函数。**

###  模板解析 
Vue 的模板是 DOM 模板，使用浏览器原生的解析器而不是自己实现一个。
相比字符串模板，DOM 模板有一些好处，但是也有问题，它必须是有效的 HTML 片段。
**一些 HTML 元素对什么元素可以放在它里面有限制。常见的限制：**
  * a 不能包含其它的交互元素（如按钮，链接）
  * ul 和 ol 只能直接包含 li
  * select 只能包含 option 和 optgroup
  * table 只能直接包含 thead, tbody, tfoot, tr, caption, col, colgroup
  * tr 只能直接包含 th 和 td
**在实际中，这些限制会导致意外的结果。**你不能依赖自定义组件在浏览器验证之前的展开结果。
例如 <my-select><option>...</option></my-select> 不是有效的模板，即使 my-select 组件最终展开为 <select>...</select>。

另一个结果是，自定义标签（包括自定义元素和特殊标签，如 <component>、<template>、 <partial> ）不能用在 ul, select, table 等对内部元素有限制的标签内。放在这些元素内部的自定义标签将被提到元素的外面，因而渲染不正确。

**对于自定义元素，应当使用 is 特性：**
<tr is="my-component"></tr>
<template> 不能用在 <table> 内，这时应使用 <tbody>，<table> 可以有多个 <tbody>：

###  使用Props选项给组件传递数据 
**组件实例的作用域是孤立的**。这意味着不能并且不应该在子组件的模板内直接引用父组件的数据。
可以使用 props 把数据传给子组件。
需要显式地用 props 选项 声明 props：
```

Vue.component('child', {
  // 声明 props
  props: ['msg'],
  // prop 可以用在模板内
  // 可以用 `this.msg` 设置
  template: '<span>![](/data/dokuwiki msg )</span>'
})

<child msg="hello!"></child>

```
HTML 特性不区分大小写。名字形式为 camelCase 的 prop 用作特性时，需要转为 kebab-case（短横线隔开）：
```

Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>![](/data/dokuwiki myMessage )</span>'
})
<!-- kebab-case in HTML -->
<child my-message="hello!"></child>

```

**Prop 绑定类型**
prop** 默认是单向绑定**：当父组件的属性变化时，将传导给子组件，但是反过来不会。这是为了防止子组件无意修改了父组件的状态——这会让应用的数据流难以理解。不过，也可以**使用 .sync 或 .once 绑定修饰符显式地强制双向或单次绑定：**
比较语法：
```

<!-- 默认为单向绑定 -->
<child :msg="parentMsg"></child>
<!-- 双向绑定 -->
<child :msg.sync="parentMsg"></child>
<!-- 单次绑定 -->
<child :msg.once="parentMsg"></child>

```
双向绑定会把子组件的 msg 属性同步回父组件的 parentMsg 属性。单次绑定在建立之后不会同步之后的变化。
注意如果 prop 是一个对象或数组，是按引用传递。在子组件内修改它会影响父组件的状态，不管是使用哪种绑定类型。

**Prop 验证**
组件可以为 props 指定验证要求。当组件给其他人使用时这很有用，因为这些验证要求构成了组件的 API，确保其他人正确地使用组件。此时 props 的值是一个对象，包含验证要求：
参考文档http://cn.vuejs.org/guide/components.html#Prop__u9A8C_u8BC1

###  父子组件通信 
子组件
this.$parent
this.$root
父组件有一个数组 this.$children，包含它所有的子元素。

**尽管可以访问父链上任意的实例，不过子组件应当避免直接依赖父组件的数据，尽量显式地使用 props 传递数据**。
另外，在子组件中修改父组件的状态是非常糟糕的做法。

**自定义事件**
Vue 实例实现了一个**自定义事件接口，用于在组件树中通信。**这个事件系统**独立于原生 DOM 事件**，用法也不同。
每个 Vue 实例都是一个事件触发器：
  * 使用 $on() 监听事件；
  * 使用 $emit() 在它上面触发事件；
  * 使用 $dispatch() 派发事件，事件沿着父链冒泡；
  * 使用 $broadcast() 广播事件，事件向下传导给所有的后代。
**不同于 DOM 事件，Vue 事件在冒泡过程中第一次触发回调之后自动停止冒泡，除非回调明确返回 true。**
```

<!-- 子组件模板 -->
<template id="child-template">
  <input v-model="msg">
  <button v-on:click="notify">Dispatch Event</button>
</template>

<!-- 父组件模板 -->
<div id="events-example">
  <p>Messages: ![](/data/dokuwiki messages | json )</p>
  <child></child>
</div>
// 注册子组件
// 将当前消息派发出去
Vue.component('child', {
  template: '#child-template',
  data: function () {
    return { msg: 'hello' }
  },
  methods: {
    notify: function () {
      if (this.msg.trim()) {
        this.$dispatch('child-msg', this.msg)
        this.msg = ''
      }
    }
  }
})

// 初始化父组件
// 将收到消息时将事件推入一个数组
var parent = new Vue({
  el: '#events-example',
  data: {
    messages: []
  },
  // 在创建实例时 `events` 选项简单地调用 `$on`
  events: {
    'child-msg': function (msg) {
      // 事件回调内的 `this` 自动绑定到注册它的实例上
      this.messages.push(msg)
    }
  }
})

```

**使用 v-on 绑定自定义事件**
上例非常好，不过从父组件的代码中不能直观的看到 "child-msg" 事件来自哪里。
如果我们**在模板中子组件用到的地方声明事件处理器会更好。为此子组件可以用 v-on 监听自定义事件**：
```

<child v-on:child-msg="handleIt"></child>

```
这样就很清楚了：当子组件触发了 "child-msg" 事件，父组件的 handleIt 方法将被调用。所有影响父组件状态的代码放到父组件的 handleIt 方法中；子组件只关注触发事件。

###  子组件索引 
尽管有 props 和 events，但是有时仍然需要在 JavaScript 中直接访问子组件。为此可以使用 **v-ref** 为子组件指定一个索引 ID。例如：
```

<div id="parent">
  <user-profile v-ref:profile></user-profile>
</div>
var parent = new Vue({ el: '#parent' })
// 访问子组件
var child = parent.$refs.profile
v-ref 和 v-for 一起用时，ref 是一个数组或对象，包含相应的子组件。

```
###  使用 Slot 分发内容与编译作用域 
在使用组件时，常常要像这样组合它们：
```

<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>

```
注意两点：
<app> 组件不知道它的挂载点会有什么内容，挂载点的内容是由 <app> 的父组件决定的。
<app> 组件很可能有它自己的模板。
**为了让组件可以组合，我们需要一种方式来混合父组件的内容与子组件自己的模板。这个处理称为内容分发**（或 “transclusion”，如果你熟悉 Angular）。使用特殊的 **<slot> 元素**作为原始内容的插槽。

` 编译作用域：父组件模板的内容在父组件作用域内编译；子组件模板的内容在子组件作用域内编译 `
分发内容是在父组件作用域内编译。
**一个常见错误是试图在父组件模板内将一个指令绑定到子组件的属性/方法**：
```

<!--如果someChildProperty 是子组件的属性，则下面这样是无效的 -->
<child-component v-show="someChildProperty"></child-component>

```
**假定 someChildProperty 是子组件的属性，上例不会如预期那样工作**。父组件模板不应该知道子组件的状态。
如果要绑定子组件内的指令到一个组件的根节点，应当在它的模板内这么做：
```

Vue.component('child-component', {
  // 有效，因为是在正确的作用域内
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})

```

单个 Slot
父组件的内容将被抛弃，除非子组件模板包含 <slot>。
```

假定 my-component 组件有下面模板：
<div>
  <h1>This is my component!</h1>
  <slot>
    如果没有分发内容则显示我。
  </slot>
</div>
父组件模板：
<my-component>
  <p>This is some original content</p>
  <p>This is some more original content</p>
</my-component>
渲染结果：
<div>
  <h1>This is my component!</h1>
  <p>This is some original content</p>
  <p>This is some more original content</p>
</div>

```

**具名 Slot**
```

例如，假定我们有一个 multi-insertion 组件，它的模板为：
<div>
  <slot name="one"></slot>
  <slot></slot>
  <slot name="two"></slot>
</div>
父组件模板：
<multi-insertion>
  <p slot="one">One</p>
  <p slot="two">Two</p>
  <p>Default A</p>
</multi-insertion>
渲染结果为：

<div>
  <p slot="one">One</p>
  <p>Default A</p>
  <p slot="two">Two</p>
</div>

```
**在组合组件时，内容分发 API 是非常有用的机制。**

###  动态组件 
多个组件可以使用同一个挂载点，然后动态地在它们之间切换。**使用保留的 <component> 元素，动态地绑定到它的 is 特性**：
```

new Vue({
  el: 'body',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
<component :is="currentView">
  <!-- 组件在 vm.currentview 变化时改变 -->
</component>

```
**keep-alive**
如果把切换出去的组件保留在内存中，可以保留它的状态或避免重新渲染。为此可以添加一个 keep-alive 指令参数：
```

<component :is="currentView" keep-alive>
  <!-- 非活动组件将被缓存 -->
</component>

```

**activate 钩子**
在切换组件时，切入组件在切入前可能需要进行一些**异步操作**。为了控制组件切换时长，给切入组件添加 activate 钩子：
```

Vue.component('activate-example', {
  activate: function (done) {
    var self = this
    loadDataAsync(function (data) {
      self.someData = data
      done()
    })
  }
})

```
注意 activate 钩子只作用于动态组件切换或静态组件初始化渲染的过程中，不作用于使用实例方法手工插入的过程中。

**transition-mode**
transition-mode 特性用于指定两个动态组件之间如何过渡。
在默认情况下，进入与离开平滑地过渡。这个特性可以指定另外两种模式：
  * in-out：新组件先过渡进入，等它的过渡完成之后当前组件过渡出去。
  * out-in：当前组件先过渡出去，等它的过渡完成之后新组件过渡进入。
示例：
```

<!-- 先淡出再淡入 -->
<component
  :is="view"
  transition="fade"
  transition-mode="out-in">
</component>
.fade-transition {
  transition: opacity .3s ease;
}
.fade-enter, .fade-leave {
  opacity: 0;
}

```

###  异步组件 
在大型应用中，我们可能需要将应用拆分为小块，只在需要时才从服务器下载。
为了让事情更简单，**Vue.js 允许将组件定义为一个工厂函数，动态地解析组件的定义。**
Vue.js 只在组件需要渲染时触发工厂函数，并且把结果缓存起来，用于后面的再次渲染。例如：
```

Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})

```
**工厂函数接收一个 resolve 回调，在收到从服务器下载的组件定义时调用。也可以调用 reject(reason) 指示加载失败。**这里 setTimeout 只是为了演示。
**怎么获取组件完全由你决定。推荐配合使用[ Webpack](https://github.com/webpack/webpack) 的代码分割功能：**
```

Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 require 语法告诉 webpack
  // 自动将编译后的代码分割成不同的块，
  // 这些块将通过 ajax 请求自动下载。
  require(['./my-async-component'], resolve)
})

```

##  深入响应式原理 
http://cn.vuejs.org/guide/reactivity.html

