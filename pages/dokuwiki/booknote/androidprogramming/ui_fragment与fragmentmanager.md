title: ui_fragment与fragmentmanager 

#  Chapter7 UI Fragment与FragmentManager 
Created Sunday 15 March 2015

问题：想要在运行时进行视图切换
解决：引入Fragment

#  fragment引入 
采用fragment而不是activity进行应用的UI管理。可以绕开Android系统activity规则的限制。
fragment是一种控制器对象。activity委派它管理用户界面。这个用户界面可以是屏幕的全部或者一部分。
管理用户界面的fragment又称为UI fragment。他又自己产生于布局文件的视图。
activity视图含有可供fragment视图插入的位置。如果有多个fragment要插入，则提供多个位置。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084603.png)
注意：fragment本身不具有在屏幕上显示视图的能力，只有将它的视图放置在activity的视图层级结构中，fragment视图才能显示在屏幕上。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084607.png)

##  Fragment与支持库 
android-support-v4
Android4.0以前的版本需要将Activity继承FragmentActivity才有管理Fragment的能力。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084623.png)

#  托管UI Fragment 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084627.png)

##  fragment生命周期 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084631.png)
fragment生命周期方法由Activity管理。而Activity的生命周期方法由系统管理。

##  托管的两种方式 
添加fragment到activity的布局中：简单但不灵活。无法切换fragment视图
代码方式：灵活。可以在运行时控制fragment的方式。推荐

##  定义容器视图 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084640.png)

##  创建UI fragment 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084659.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084714.png)
第三个参数告知布局生产器是否将生成的视图添加给父视图。这里我们传入了false参数，因为我们将通过activity代码的方式添加视图。

#  添加UI Fragment到FragmentManager 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084724.png)
FragmentManager类具体的管理是：
  * Fragment队列；
  * FragmentTransaction事务的回退栈
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084739.png)

##  Fragment事务 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084748.png)
add（）方法是整个事务的核心，包含两个参数，即容器视图资源ID和新创建的Fragment对象。
容器视图资源ID主要有两点作用：
  * 提供插入点；
  * 是Fragment在队列中的唯一标志。
为什么队列中已经有Fragment存在了呢？因设备旋转或者内存吃紧而使Activity被销毁后重新创建时。fragment队列会被重建。从而恢复到原来的状态。所以在创建新对象之前，检查队列中是否已经存在。

##  FragmentManger与Fragment生命周期 

##  Activity使用fragment的理由 
AUF（Always use Fragments）


