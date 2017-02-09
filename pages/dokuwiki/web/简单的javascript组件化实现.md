title: 简单的javascript组件化实现 

#  简单的JavaScript组件化实现 
作为一名前端菜鸟，最近看react例子，根据理解自己也简单实现了一下组件的继承和事件机制。
原始的组件写法
```

(function($) {
  $.pluginName = function(element, options) {
    var defaults = {
      title: '',
      content: '',
      showOKBtn: 1, // 显示确定按钮
      showCCBtn: 1, // 显示取消按钮
      onFoo: function() {} // callback
    }
    var plugin = this;
    plugin.settings = {}
    var $element = $(element);
    plugin.init = function(options) {
      this.settings = $.extend({}, defaults, options);
      this.initNode(options);
    }
    // public method.
    plugin.show = function() {
      // ...
    }
    plugin.hide = function() {
      // ...	
    }
    plugin.initNode = function(options) {
      var $okBtn = $element.find(''),
        $content = $element.find('');
      // ....
      // 部分逻辑
      $content.text(plugin.settings.content);
      $okBtn.on('click', $.proxy(this.onOk, this));
    }
    plugin.onOk = function(){
      this.hide();
      plugin.settings.onFoo();
    }
    plugin.init();
  }
  $.fn.pluginName = function(options) {
    return this.each(function() {
      if (undefined == $(this).data('pluginName')) {
        var plugin = new $.pluginName(this, options);
        $(this).data('pluginName', plugin);
      }
    });
  }
})(jQuery);
// 使用
var template = '<div>...弹框html...</div>';
$(template).pluginName({
  content: '确定删除该地址'
}).show();

```
一般我们写得入门级jquery组件，基本就是这样一个模板。
这里我们实现了一个基本的弹窗组件，也完成了需求方的要求，oh ye！
![](/data/dokuwiki/web/pasted/20151014-132747.png)

处理一套风格相似的组件的时候，**通过传递不同的参数来控制不同的ui显示和逻辑代码执行**，确实可以解决问题，随着功能的一步步增加，这个组件就变得越来越臃肿，代码耦合成度变高，到最后自己都搞不清楚每个参数不同值代表的意思。况且在团队中都是多个人维护同一个组件，这简直就是一场悲剧。

**继承**
这时候面向对象的思维就出场了
![](/data/dokuwiki/web/pasted/20151014-133844.png)
我们发现设置titile， 关闭窗体是大家共有的功能。这里可以抽象成一个基础组件，新的组件继承这个组件即可。
```

(function() {
    var BaseWindon = Kclass.extend({
  init: function(options){
     //公共功能 
      $titleElm.text(options.title);
      $closeElm.on('click', $.proxy(this.close, this));
  },
        // 销毁
  destroy: function(){
  }
  close: function(){
  }
    };
    return BaseWindon;
})()
 在子类中 require('BaseWindon');
 (function() {
    var AddAddressWindon = BaseWindon.extend({
  init: function(options){
      // 调用parent的init
      this.supr();
  },
  validate: function(){
  },
  // 组件自己的功能
  submit: function(){
  }
    };
    return AddAddressWindon;
})()

```
    

**事件机制**
事件机制对应着一种设计模式-观察者模式。
```

(function() {
  var win = new BaseWindow();
  win.on('ok', function(){
    console.log('on ok!');
  });
  win.emit('ok');  
  // log  --- on ok!
})()


```
显然事件驱动的方式更加优雅，相比之下第一种手动触发callback的方式显得有点out。事件机制的方式，在监听on和触发emit的时机上也显得更加灵活，对于只需要触发一次的callback，你只需要用once函数来监听。写代码好像突然变得好爽好舒服。
最后记得提供一个销毁组件的方法，一个简单的组件就完成了。
当然要更好的组件还需要提供
模板机制
双向绑定
参考：http://www.tuicool.com/articles/rYf2Yji