title: 使用listfragment显示列表 

#  Chapter9 使用ListFragment显示列表 
Created Wednesday 18 March 2015

![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084944.png)

#  单例与数据集中存储 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084948.png)
为保证单例总是有Context可用，我们需要使用ApplicationContext

#  抽象Activity类 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085001.png)
#  ListFragment、ListView以及ArrayAdapter 
ListView只是当需要显示某个列表项时才通过Adapter申请一个可用的视图对象。Adapter是一个控制器，从模型中获取数据，并将其提供给ListView显示。
adapter负责：
  * 创建必要的视图对象；
  * 用模型曾数据填充视图对象；
  * 将准备好的视图返回给ListView
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085033.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085046.png)
##  创建ArrayAdapter<T>实例 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085101.png)
注意：ArrayAdapter<T>.getView(...)默认实现方法依赖于数据集中对象的toString方法。所以使用默认行为时记得覆盖toString方法。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085109.png)
convertView参数是一个已存在的列表项。adapter可以从新配置并返回它，当然也可能不存在为null。
故而需要检测是否为null.这样做的目的是为了复用对象，提高效率。


