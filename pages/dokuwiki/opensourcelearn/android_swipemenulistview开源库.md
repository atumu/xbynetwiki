title: android_swipemenulistview开源库 

#  Android_SwipeMenuListView开源库 
使用过程中问题多多：
1.不要再项目中将其添加为依赖，而是应该直接将代码复制到项目中，否则会出现各种问题。
2.由于SwipeMenuListView在内部对传入的Adapter通过WrapperListAdapter进行了包装。所以通过swipelistView.getAdapter进行强制转换为出现问题。
对于Adapter进行转换需要这样```

(MyAdapter)(((WrapperListAdapter)swipelistView.getAdapter()).getWrappedAdapter())

```