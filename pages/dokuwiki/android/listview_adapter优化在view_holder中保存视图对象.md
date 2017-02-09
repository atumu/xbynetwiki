title: listview_adapter优化在view_holder中保存视图对象 

#  listview_adapter优化在view_holder中保存视图对象 
**使用范围：覆盖Adapter的getView方法的时候。利用getView方法的convertView参数。**

你的代码可能在滑动ListView时频繁地调用findViewById()，而这可使效果变慢。即使在Adapter为了回收而返回一个已经展现出来的视图，你仍然需要查找这些元素并且更新他们。一个循环使用findViewById()的方法是使用“view holder”设计模式。
一个findViewById()对象存储布局内的每个组建视图的标记域，你可以立即访问而不需要反复的查询他们。首先，你需要建立一个类来保存具体的视图。例如：1
**利用convertView就是利用Android的Recycler机制，利用convertView来重新回收view，效率就有了本质的提高。
ViewHolder将需要缓存的view封装好,convertView的setTag才是将这些缓存起来供下次调用。**
```

static class ViewHolder {
  TextView text;
  TextView timestamp;
  ImageView icon;
  ProgressBar progress;
  int position;}

```
然后填充findViewById()并且在布局中保存它。
```

@Override
public View getView(int position,View convertView, ViewGroup parent){
  if(convertView==null){
	holder = new ViewHolder();
      convertView=inflater.inflate(R.layout_list_item,null,false);
	holder.icon = (ImageView) convertView.findViewById(R.id.listitem_image);
	holder.text = (TextView) convertView.findViewById(R.id.listitem_text);
	holder.timestamp = (TextView) convertView.findViewById(R.id.listitem_timestamp);
	holder.progress = (ProgressBar) convertView.findViewById(R.id.progress_spinner);
	convertView.setTag(holder);
  }else{
      holder=(ViewHolder)covertView.getTag();
    }
  ....
}

```
现在你可以轻松的访问每一个视图而不需要频繁的去查询他们，这节省了宝贵的处理器周期。