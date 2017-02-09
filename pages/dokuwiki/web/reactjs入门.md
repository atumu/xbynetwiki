title: reactjs入门 

#  ReactJS组件化构建用户界面 
官网：http://facebook.github.io/react/
中文网站：http://www.reactjs.cn/
建议参考教程：
http://www.ruanyifeng.com/blog/2015/03/react.html
http://www.cocoachina.com/webapp/20150721/12692.html

**目前API还不稳定，还没有到1.0版本，不建议使用该组件**
##  概述 
**特点**
仅仅是UI
许多人使用React作为MVC架构的V层。 尽管React并没有假设过你的其余技术栈， 但它仍可以作为一个小特征轻易地在已有项目中使用

虚拟DOM
React为了更高超的性能而使用虚拟DOM作为其不同的实现。 它同时也可以由服务端Node.js渲染 － 而不需要过重的浏览器DOM支持

数据流
React实现了单向响应的数据流，从而减少了重复代码，这也是它为什么比传统数据绑定更简单。

**一个简单的组件**
React组件通过一个 ` render() ` 方法**，接受输入的参数并返回展示的对象。** 
以下这个例子使用了JSX，它类似于XML的语法
**输入的参数通过 render() 传入组件后，将存储在` this.props `**

` JSX是可选的 `，并不强制要求使用。
```

var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

React.render(<HelloMessage name="John" />, mountNode);

```

**一个有状态的组件**
除了` 接受输入数据（通过 this.props ） `，组件还可以` 保持内部状态数据（通过 this.state ） `。当一个组件的状态数据的变化，展现的标记将被重新调用`  render() 更新 `。
```

var Timer = React.createClass({
  getInitialState: function() {
    return {secondsElapsed: 0};
  },
  tick: function() {
    this.setState({secondsElapsed: this.state.secondsElapsed + 1});
  },
  componentDidMount: function() {
    this.interval = setInterval(this.tick, 1000);
  },
  componentWillUnmount: function() {
    clearInterval(this.interval);
  },
  render: function() {
    return (
      <div>Seconds Elapsed: {this.state.secondsElapsed}</div>
    );
  }
});

React.render(<Timer />, mountNode);

```
这里，我们使用到了一个方法` getInitialState `,**这个函数在组件初始化的时候执行，必需返回NULL或者一个对象。**这里我们可以通过` this.state.属性名 `来访问属性值，当要修改这个属性值时，要使用` setState `方法。我们声明tick方法，来绑定到button上面，实现改变state.enable的值。
当用户点击组件，导致状态变化，this.setState 方法就修改状态值，**每次修改以后，自动调用 this.render 方法，再次渲染组件**。
这里值得注意的几点如下：
1）getInitialState函数必须有返回值，可以是NULL或者一个对象。
2）访问state的方法是this.state.属性名。
3）**变量用{}包裹，不需要再加双引号。**

**组件的生命周期**
组件的生命周期分成三个状态：
  * Mounting：已插入真实 DOM
  * Updating：正在被重新渲染
  * Unmounting：已移出真实 DOM
React 为每个状态都提供了两种处理函数，**will 函数在进入状态之前调用，did 函数在进入状态之后调用**，三种状态共计五种处理函数。
  * componentWillMount()
  * componentDidMount()
  * componentWillUpdate(object nextProps, object nextState)
  * componentDidUpdate(object prevProps, object prevState)
  * componentWillUnmount()
此外，React 还提供两种特殊状态的处理函数。
  * componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用
  * shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用

''组件名称首字母必须大写。
变量名用{}包裹，且不能加双引号。''
##  快速入门 
现在最热门的前端框架有AngularJS、React、Bootstrap等。自从接触了ReactJS，ReactJs的**虚拟DOM（Virtual DOM）和组件化的开发**深深的吸引了我
虚拟DOM(virtual-dom)不仅带来了简单的UI开发逻辑，同时也带来了组件化开发的思想，所谓组件，即封装起来的具有独立功能的UI部件。React推荐以组件的方式去重新思考UI构成，将UI上每一个功能相对独立的模块定义成组件，然后将小的组件通过组合或者嵌套的方式构成大的组件，最终完成整体UI的构建。
React认为一个组件应该具有如下特征：
（1）可组合（Composeable）：一个组件易于和其它组件一起使用，或者嵌套在另一个组件内部。如果一个组件内部创建了另一个组件，那么说父组件拥有（own）它创建的子组件，通过这个特性，一个复杂的UI可以拆分成多个简单的UI组件；
（2）可重用（Reusable）：每个组件都是具有独立功能的，它可以被使用在多个UI场景；
（3）可维护（Maintainable）：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护；

```

<!DOCTYPE html>
<html>
  <head>
    <script src="build/react.js"></script>
    <script src="build/JSXTransformer.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      React.render(
        <h1>Hello, world!</h1>,
        document.getElementById('example')
      );
    </script>
  </body>
</html>

```
这里需要注意的是，react并不依赖jQuery，当然我们可以使用jQuery` ，但是render里面第二个参数必须使用JavaScript原生的getElementByID方法，不能使用jQuery来选取DOM节点。 `
在 JavaScript 代码里写着 XML 格式的代码称为 ` JSX `；可以去 JSX 语法 里学习更多 JSX 相关的知识。为了把 JSX 转成标准的 JavaScript，我们用 <script type="` text/babel `"> 标签包裹着含有 JSX 的代码，然后引入`  JSXTransformer.js ` 库来实现在浏览器里的代码转换。

