title: jquery_noty 

#  jQuery.noty消息提醒插件 
官网：http://ned.im/noty/#/about
jQuery noty gitHub：https://github.com/needim/noty
中文参考资料：http://segmentfault.com/a/1190000000372018
noty是一个jQuery插件，可以很方便的创建以下几种消息：alert（弹出框消息）、 success（成功消息）、error（错误消息）、warning（警告消息）、information（提示 消息）、confirmation（确认消息）
**每一个通知信息，都会添加到一个消息队列中。**
通知信息支持以下几种位置：top（正上）topLeft（左上） topCenter （中上） topRight（右上） center （页面中间） centerLeft （中间居左） centerRight （中间居右） bottom （页面底部） bottomLeft （底部居左） bottomCenter （底部居中） bottomRight（底部局右）
通过给出的API可以定制noty的文本（可以是HTML）、动画效果、显示的速度、可以定制按钮
**提供回调函数，**在点击确定等按钮的时候可以使用
```

<script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.7/jquery.min.js"></script>
<script type="text/javascript" src="js/noty/packaged/jquery.noty.packaged.min.js"></script>

```
##  选项列表 

```

$.noty.defaults = {
    layout: 'top', //top - topLeft - topCenter - topRight - center - centerLeft - centerRight - bottom - bottomLeft - bottomCenter - bottomRight
    theme: 'defaultTheme', // or 'relax'
    type: 'alert', //alert - success - error - warning - information - confirmation
    text: '', // can be html or string
    dismissQueue: true, // If you want to use queue feature set this true
    template: '<div class="noty_message"><span class="noty_text"></span><div class="noty_close"></div></div>',
    animation: {
        open: {height: 'toggle'}, // or Animate.css class names like: 'animated bounceInLeft'
        close: {height: 'toggle'}, // or Animate.css class names like: 'animated bounceOutLeft'
        easing: 'swing',
        speed: 500 // opening & closing animation speed
    },
    timeout: false, // delay for closing event. Set false for sticky notifications
    force: false, // adds notification to the beginning of queue when set to true
    modal: false,
    maxVisible: 5, // you can set max visible notification for dismissQueue true option,
    killer: false, // for close all notifications before show
    closeWith: ['click'], // ['click', 'button', 'hover', 'backdrop'] // backdrop click will close all notifications
    callback: {
        onShow: function() {},
        afterShow: function() {},
        onClose: function() {},
        afterClose: function() {},
        onCloseClick: function() {},
    },
    buttons: false // an array of buttons
};

```
```

var n = $('.custom_container').noty({text: 'NOTY - a jquery notification library!'});

```

##  主题 
主题noty已经内容了noty/themes/default.js文件中，可以自己修改这个文件，然后使用这个主题属性。
```

<script type="text/javascript" src="js/noty/themes/your_new_theme.js"></script>
var n = noty({
    text: 'NOTY - a jquery notification library!',
    theme: 'your_new_theme',
});

```
##  按钮 
```

noty({
	text: 'Do you want to continue?',
	buttons: [
		{addClass: 'btn btn-primary', text: 'Ok', onClick: function($noty) {

				// this = button element
				// $noty = $noty element

				$noty.close();
				noty({text: 'You clicked "Ok" button', type: 'success'});
			}
		},
		{addClass: 'btn btn-danger', text: 'Cancel', onClick: function($noty) {
				$noty.close();
				noty({text: 'You clicked "Cancel" button', type: 'error'});
			}
		}
	]
});

```
##  回调函数 
onShow - afterShow - onClose - afterClose - afterCloseClick
##  API 
$.noty.get(id) - Returns a NOTY javascript object
$.noty.close(id) - Close a NOTY - same as var n = noty({text: 'Hi!'})); n.close();
$.noty.clearQueue() - Clears the notification queue
$.noty.closeAll() - Close all notifications
$.noty.setText(id, text) - Notification text updater - same as var n = noty({text: 'Hi!'})); n.setText('Hi again!');
$.noty.setType(id, type) - Notification type updater - same as var n = noty({text: 'Hi!'})); n.setType('error');

