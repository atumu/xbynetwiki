title: chapter10游戏与动画 

游戏与动画
=====
Created Sunday 10 January 2010

Android支持OpenGL ES
Android游戏框架：
AndEngine 开源免费
Box2D开源免费
Corona SDK 收费
Flixel开源免费
libgdx开源免费
PlayN开源免费。
rokon开源免费
Unity 收费

处理定时键盘输入
--------
Handler的sendMessageDelayed延时发送消息。
```

class MyHandler extends Handler{
	@Override
	public void handleMessage(Message msg){
		doTask1();
	}
	//在按键处理监听器中调用该方法可以延时执行doTask1();
	public void sleep(long timeInterval){
		this.removeMessage(0);
		sendMessageDelayed(obtainMessage(0),timeInterval);
	}
}

```