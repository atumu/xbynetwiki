title: 使用viewpager 

#  Chapter11 使用ViewPager 
Created Wednesday 18 March 2015

![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085306.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085309.png)

##  以代码方式定义并产生布局 
FragmentManager要求任何作为Fragment容器的视图必须具有资源ID（可以是自定义的ID资源）。
ViewPager是一个fragment容器。因此必须赋予其资源ID：
独立资源ID: [[/res/values/mid.xml:]]
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085319.png)
创建ViewPager内容视图：
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085329.png)
注意：ViewPager类来自于支持库，而在标准库中是没有的。

##  ViewPager与PagerAdapter 
ViewPager与PagerAdaper有点类似与AdapterView与Adapter的关系。
我们可以是使用PagerAdapter的两个子类工作FragmentPagerAdapter、FragmentStatePagerAdapter.
先来介绍FragmentStatePagerAdapter
我们主要覆盖其两个方法:getCount()以及getItem(int)该方法返回用于显示的Fragment对象。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085343.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085347.png)
注意到一个问题，那就是ViewPager默认只显示PagerAdaper中的第一个列表项。解决方案如下：
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085352.png)
还可以添加监听器:ViewPager.OnPageChangeListener
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085355.png)

##  FragmentStatePagerAdapter与FragmentPagerAdapter 
两者的唯一区别就是在卸载不需要的fragment的处理方式不一样。
前者采用保存状态并销毁的机制，适用于fragment很多的情形
后者采用detach(Fragment)的机制销毁视图保留对象的机制，适用于fragment很少的情形。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085403.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-085406.png)

#  深入学习：ViewPager的工作原理 
略。


