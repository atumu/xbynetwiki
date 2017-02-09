title: android官方文档学习之layout 

#  android官方文档学习之Layout 
##  布局优化原则 
在Android UI布局过程中，通过遵守一些惯用、有效的布局原则，我们可以制作出高效且复用性高的UI，概括来说包括如下几点：
  * 尽量多使用` RelativeLayout和LinearLayout `, 不要使用绝对布局AbsoluteLayout，在布局层次一样的情况下， 建议使用LinearLayout代替RelativeLayout, 因为LinearLayout性能要稍高一点，但往往RelativeLayout可以简单实现LinearLayout嵌套才能实现的布局。
  * 将可复用的组件抽取出来并通过` include `标签使用；
  * 使用` ViewStub `标签来加载一些不常用的布局；
  * 使用` merge `标签减少布局的嵌套层次；
##  android:gravity / android:layout_gravity区别： 
  * android:gravity　是设置该view里面的内容相对于该view的位置，例如设置button里面的text相对于view的靠左，居中等位置。(也可以在Layout布局属性中添加，设置Layout中组件的位置)
  * android:layout_gravity 是用来设置该view相对与父view的位置，例如设置button在layout里面的相对位置：屏幕居中，水平居中等。 即android:gravity用于设置View中内容相对于View组件的对齐方式，而android:layout_gravity用于设置View组件相对于Container的对齐方式。 说的再直白点，就是android:gravity只对该组件内的东西有效，android:layout_gravity只对组件自身有效 
##  FrameLayout 
FrameLayout是最简单的一个布局对象。它被定制为你屏幕上的一个空白备用区域，之后你可以在其中填充一个单一对象 — 比如，一张你要发布的图片。所有的子元素将会固定在屏幕的左上角；你不能为FrameLayout中的一个子元素指定一个位置。后一个子元素将会直接在前 一个子元素之上进行覆盖填充，把它们部份或全部挡住（除非后一个子元素是透明的）。
```

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >

 <!-- 我们在这里加了一个Button按钮 --> 
<Button 
    android:text="button" 
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
/> 
<TextView 
    android:text="textview"
    android:textColor="#0000ff" 
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
/> 
</FrameLayout>

```
![](/data/dokuwiki/android/pasted/20150608-171150.png)

##  LinearLayout 
LinearLayout以你为它设置的垂直或水平的属性值，来排列所有的子元素。所有的子元素都被堆放在其它元素之后，因此一个垂直列表的每一行只会有 一个元素，而不管他们有多宽，而一个水平列表将会只有一个行高（高度为最高子元素的高度加上边框高度）。LinearLayout保持子元素之间的间隔以 及互相对齐（相对一个元素的右对齐、中间对齐或者左对齐）。

LinearLayout还支持为单独的子元素指定**layout_weight** 。好处就是允许子元素可以填充屏幕上的剩余空间。这也避免了在一个大屏幕中，一串小对象挤 成一堆的情况，而是允许他们放大填充空白。子元素指定一个weight 值，剩余的空间就会按这些子元素指定的weight 比例分配给这些子元素。默认的 weight 值为0。例如，如果有三个文本框，其中两个指定了weight 值为1，那么，这两个文本框将等比例地放大，并填满剩余的空间，而第三个文本框 不会放大。

android:orientation
```

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:orientation="vertical" 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
   <LinearLayout 
    android:orientation="vertical" 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:layout_weight="2"> 
    <TextView
        android:text="Welcome to Mr Wei's blog" 
        android:textSize="15pt"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
     /> 
    </LinearLayout> 
    <LinearLayout 
    android:orientation="horizontal" 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:layout_weight="1"> 
    
    <TextView 
        android:text="red"

        android:gravity="center_horizontal" //这里字水平居中
        android:background="#aa0000"
        android:layout_width="wrap_content"
        android:layout_height="fill_parent"
        android:layout_weight="1"/> 
    <TextView 
        android:text="green"
        android:gravity="center_horizontal "
        android:background="#00aa00"
        android:layout_width="wrap_content"
        android:layout_height="fill_parent"
        android:layout_weight="1"/>    
    </LinearLayout>
</LinearLayout>

```
![](/data/dokuwiki/android/pasted/20150608-171350.png)
##  AbsoluteLayout: 
AbsoluteLayout 可以让子元素指定准确的x/y坐标值，并显示在屏幕上。(0, 0)为左上角，当向下或向右移动时，坐标值将变大。AbsoluteLayout 没有页边框，允许元素之间互相重叠（尽管不推荐）。我们通常不推荐使用 AbsoluteLayout ，除非你有正当理由要使用它，因为它使界面代码太过刚性，以至于在不同的设备上可能不能很好地工作。
```

<?xml version="1.0" encoding="utf-8"?>
<AbsoluteLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >
<EditText 
    android:text="Welcome to Mr Wei's blog"
    android:layout_width="fill_parent" 
    android:layout_height="wrap_content" 
/> 
<Button 
    android:layout_x="250px" //设置按钮的X坐标
    android:layout_y="40px" //设置按钮的Y坐标
    android:layout_width="70px" //设置按钮的宽度
    android:layout_height="wrap_content"
    android:text="Button" 
/> 
</AbsoluteLayout>

```
![](/data/dokuwiki/android/pasted/20150608-171557.png)
##  RelativeLayout 
RelativeLayout 允许子元素指定他们相对于其它元素或父元素的位置（通过ID 指定）。因此，你可以以右对齐，或上下，或置于屏幕中央的形式来 排列两个元素。元素按顺序排列，因此如果第一个元素在屏幕的中央，那么相对于这个元素的其它元素将以屏幕中央的相对位置来排列。如果使用XML 来指定这个 layout ，在你定义它之前，被关联的元素必须定义。

RelativeLayout相关的**布局属性**:
  * android:layout_above 将该控件置于给定ID的控件之上
  * android:layout_below 将该控件的置于给定ID控件之下
  * android:layout_toLeftOf 将该控件置于给定ID的控件之左
  * android:layout_toRightOf 将该控件置于给定ID的控件之右

RelativeLayout布局中相对于父控件来说**位置属性**：
  * android:layout_alignParentLeft 如果为True，该控件位于父控件的左部
  * android:layout_alignParentRight 如果为True，该控件位于父控件的右部
  * android:layout_alignParentTop 如果为True，该控件位于父控件的顶部
  * android:layout_alignParentBottom 如果为True，该控件位于父控件的底部

RelativeLayout布局时**对齐相关的属性**：
  * android:layout_alignBaseline 该控件基线对齐给定ID的基线
  * android:layout_alignBottom 该控件于给定ID的控件底部对齐
  * android:layout_alignLeft 该控件于给定ID的控件左对齐
  * android:layout_alignRight 该控件于给定ID的控件右对齐
  * android:layout_alignTop 该控件于给定ID的控件顶对齐
  * android:layout_centerHorizontal 如果为True，该控件将被置于水平方向的中央
  * android:layout_centerInParent 如为Ture，该控件将被置于父控件水平方向和垂直方向
  * android:layout_centerVertical 如果为True，该控件将被置于垂直方向的中央

这仅仅是几个例子，所有的布局属性我们可以在RelativeLayout.LayoutParams中找到。
```

 <?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <TextView 
        android:id="@+id/label" 
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="Welcome to Mr Wei's blog:"/> 
    <EditText 
        android:id="@+id/entry" 
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/label"/> 
    <Button 
        android:id="@+id/ok" 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/entry" 
        android:layout_alignParentRight="true"
        android:layout_marginLeft="10dip"
        android:text="OK" /> 
    <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toLeftOf="@id/ok"
        android:layout_alignTop="@id/ok"
        android:text="Cancel" /> 
</RelativeLayout>

```
![](/data/dokuwiki/android/pasted/20150608-171855.png)
##  TableLayout: 
TableLayout 将子元素的位置分配到行或列中。**一个TableLayout 由许多的TableRow 组成**，每个TableRow 都会定义一个 row （事实上，你可以定义其它的子对象，这在下面会解释到）。TableLayout 容器不会显示row 、cloumns 或cell 的边框线。每个 row 拥有0个或多个的cell ；每个cell 拥有一个View 对象。
在TableRow中的单元格可以为empty，并且通过` android:layout_column `可以设置index值（索引从0开始）实现跳开某些单元格，直接报控件放置指定索引的单元格中。
**添加View,设置layout_height以及背景色，就可以实现一条间隔线**。` android:layout_span `可以设置合并几个单元格。
TableLayout主要属性介绍:
  * android:collapseColumns：隐藏指定的列
  * android:shrinkColumns：收缩指定的列以适合屏幕，不会挤出屏幕
  * android:stretchColumns：尽量把指定的列填充空白部分
  * android:layout_column:控件放在指定的列
  * android:layout_span:该控件所跨越的列数

注意：android:shrinkColumns和android:stretchColumns的值都是以0开始的index，
但必须是string值，即用"1,2,5"来表示。可以用"*"来表示all columns。而且同一column可以同时设置为shrinkable和stretchable。

现在隐藏第二列：android:collapseColumns="1"  如果同时隐藏 第二列和第三列  可以这样：android:collapseColumns="1,2";用,逗号分割。

```

<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:collapseColumns="1" <!--影藏第二列  列的索引从0开始 -->
    tools:context=".TableLayoutOneActivity" >

```
###  TableLayout实现边框效果： 
为了醒目，需要给TableLayout设定边框来区分不同的表格中的信息：
主要是通过设定TableLayout、TableRow 、View颜色反衬出边框的颜色。
例如TableLayout的` android:layout_margin="2dip" `设置为这个数字 ，
在指定一个背景色` android:background="#00ff00" `，它里面的颜色也是这样子的设置，就可以呈现出带边框的效果了。
```

<View android:layout_height="2dip" android:background="#FF909090" /> //这里是上图中的分隔线

```
  
` android:layout_span="2" `:该控件所跨越的列数 。此空间跨越2列
```

 <TableRow>
        <Button android:text="第三行第一列"/>
        <Button android:text="第三行第二列" android:layout_span="2"/><!---此Button跨越2列。占据第二列和第三列-->
     </TableRow>

```
```

<?xml version="1.0" encoding="utf-8"?>
<TableLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="fill_parent" android:layout_height="fill_parent"
    android:stretchColumns="1">
    <TableRow> 
        <TextView android:layout_column="1" android:text="Open..." />
        <TextView android:text="Ctrl-O" android:gravity="right" />
    </TableRow> 
    <TableRow> 
        <TextView android:layout_column="1" android:text="Save..." />
        <TextView android:text="Ctrl-S" android:gravity="right" />
    </TableRow> 
    <View android:layout_height="2dip" android:background="#FF909090" /> //这里是上图中的分隔线
    <TableRow> 
        <TextView android:text="X" />
        <TextView android:text="Export..." />
        <TextView android:text="Ctrl-E" android:gravity="right " />
    </TableRow> 
    <View android:layout_height="2dip" android:background="#FF909090" /> 
    <TableRow> 
        <TextView android:layout_column="1" android:text="Quit"
            android:padding="3dip" />
    </TableRow> 
</TableLayout>

```
##  GridView 
参考：http://blog.csdn.net/hellogv/article/details/4567095
GridView是一个在可滚动的二维网格空间中显示项目的ViewGroup。元件会使用ListAdapter自动插入网格布局中。
 GridView跟ListView都是比较常用的多控件布局，而GridView更是实现九宫图的首选!本文就是介绍如何使用GridView实现九宫图。
```

<?xml version="1.0" encoding="utf-8"?>
<GridView xmlns:android="http://schemas.android.com/apk/res/android" 
    android:id="@+id/gridview"
    android:layout_width="fill_parent" 
    android:layout_height="fill_parent"
    android:columnWidth="90dp"
    android:numColumns="auto_fit"
    android:verticalSpacing="10dp"
    android:horizontalSpacing="10dp"
    android:stretchMode="columnWidth"
    android:gravity="center"
/>

```
介绍一下里面的某些属性：

android:numColumns="auto_fit" ，GridView的列数设置为自动

android:columnWidth="90dp"，每列的宽度，也就是Item的宽度
android:stretchMode="columnWidth"，缩放与列宽大小同步
android:verticalSpacing="10dp"，两行之间的边距，如：行一(NO.0~NO.2)与行二(NO.3~NO.5)间距为10dp
android:horizontalSpacing="10dp"，两列之间的边距。
接下来介绍 night_item.xml，这个XML跟前面ListView的ImageItem.xml很类似：

```

<?xml version="1.0" encoding="utf-8"?>  
<RelativeLayout   
         xmlns:android="http://schemas.android.com/apk/res/android"   
         android:layout_height="wrap_content"   
         android:paddingBottom="4dip" android:layout_width="fill_parent">  
         <ImageView   
               android:layout_height="wrap_content"   
               android:id="@+id/ItemImage"   
               android:layout_width="wrap_content"   
               android:layout_centerHorizontal="true">   
         </ImageView>  
         <TextView   
               android:layout_width="wrap_content"   
               android:layout_below="@+id/ItemImage"   
               android:layout_height="wrap_content"   
               android:text="TextView01"   
               android:layout_centerHorizontal="true"   
               android:id="@+id/ItemText">  
         </TextView>  
</RelativeLayout> 

``` 

最后就是JAVA的源代码了，也跟前面的ListView的JAVA源代码很类似，不过多了“选中”的事件处理：
```

  public void onCreate(Bundle savedInstanceState) {  
      super.onCreate(savedInstanceState);  
      setContentView(R.layout.main);  
      GridView gridview = (GridView) findViewById(R.id.gridview);  
        
      //生成动态数组，并且转入数据  
      ArrayList<HashMap<String, Object>> lstImageItem = new ArrayList<HashMap<String, Object>>();  
      for(int i=0;i<10;i++)  
      {  
        HashMap<String, Object> map = new HashMap<String, Object>();  
        map.put("ItemImage", R.drawable.icon);//添加图像资源的ID  
    map.put("ItemText", "NO."+String.valueOf(i));//按序号做ItemText  
        lstImageItem.add(map);  
      }  
      //生成适配器的ImageItem <====> 动态数组的元素，两者一一对应  
      SimpleAdapter saImageItems = new SimpleAdapter(this, //没什么解释  
                                                lstImageItem,//数据来源   
                                                R.layout.night_item,//night_item的XML实现  
                                                  
                                                //动态数组与ImageItem对应的子项          
                                                new String[] {"ItemImage","ItemText"},   
                                                  
                                                //ImageItem的XML文件里面的一个ImageView,两个TextView ID  
                                                new int[] {R.id.ItemImage,R.id.ItemText});  
      //添加并且显示  
      gridview.setAdapter(saImageItems);  
      //添加消息处理  
      gridview.setOnItemClickListener(new ItemClickListener());  
  }  
    
  //当AdapterView被单击(触摸屏或者键盘)，则返回的Item单击事件  
  class  ItemClickListener implements OnItemClickListener  
  {  
public void onItemClick(AdapterView<?> arg0,//The AdapterView where the click happened   
                                  View arg1,//The view within the AdapterView that was clicked  
                                  int arg2,//The position of the view in the adapter  
                                  long arg3//The row id of the item that was clicked  
                                  ) {  
    //在本例中arg2=arg3  
    HashMap<String, Object> item=(HashMap<String, Object>) arg0.getItemAtPosition(arg2);  
    //显示所选Item的ItemText  
    setTitle((String)item.get("ItemText"));  
}  
      
  }  

```
![](/data/dokuwiki/android/pasted/20150608-174557.png)
##  ListView 
参考：http://blog.csdn.net/hellogv/article/details/4542668
ListView是一个经常用到的控件，ListView里面的每个子项Item可以使一个字符串，也可以是一个组合控件。先说说ListView的实现：
1.准备ListView要显示的数据 ；
2.使用 一维或多维 动态数组 保存数据；
2.构建适配器 ， 简单地来说， 适配器就是 Item数组 ， 动态数组 有多少元素就生成多少个Item；
3.把 适配器 添加到ListView,并显示出来。
my_listitem.xml的代码如下，my_listitem.xml用于设计ListView的Item：

```

<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout   
        android:layout_width="fill_parent"   
        xmlns:android="http://schemas.android.com/apk/res/android"   
        android:orientation="vertical"  
        android:layout_height="wrap_content"   
        android:id="@+id/MyListItem"   
        android:paddingBottom="3dip"   
        android:paddingLeft="10dip">  
        <TextView   
                android:layout_height="wrap_content"   
                android:layout_width="fill_parent"   
                android:id="@+id/ItemTitle"   
                android:textSize="30dip">  
        </TextView>  
        <TextView   
                android:layout_height="wrap_content"   
                android:layout_width="fill_parent"   
                android:id="@+id/ItemText">  
        </TextView>  
</LinearLayout> 

``` 
```

public void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    setContentView(R.layout.main);  
    //绑定XML中的ListView，作为Item的容器  
    ListView list = (ListView) findViewById(R.id.MyListView);  
      
    //生成动态数组，并且转载数据  
    ArrayList<HashMap<String, String>> mylist = new ArrayList<HashMap<String, String>>();  
    for(int i=0;i<30;i++)  
    {  
        HashMap<String, String> map = new HashMap<String, String>();  
        map.put("ItemTitle", "This is Title.....");  
        map.put("ItemText", "This is text.....");  
        mylist.add(map);  
    }  
    //生成适配器，数组===》ListItem  
    SimpleAdapter mSchedule = new SimpleAdapter(this, //没什么解释  
      mylist,//数据来源   
       R.layout.my_listitem,//ListItem的XML实现  
      //动态数组与ListItem对应的子项          
      new String[] {"ItemTitle", "ItemText"},   
       //ListItem的XML文件里面的两个TextView ID  
       new int[] {R.id.ItemTitle,R.id.ItemText});  
    //添加并且显示  
    list.setAdapter(mSchedule);  
}  

```

```

<?xml version="1.0" encoding="utf-8"?>    
<RelativeLayout     
         android:layout_width="fill_parent"     
         xmlns:android="http://schemas.android.com/apk/res/android"     
         android:layout_height="wrap_content"     
         android:paddingBottom="4dip"     
         android:paddingLeft="12dip">    
         <ImageView     
               android:layout_width="wrap_content"     
               android:id="@+id/itemImage" android:layout_height="fill_parent">     
         </ImageView>    
         <TextView     
               android:text="TextView01"     
               android:layout_height="wrap_content"     
               android:layout_width="fill_parent"     
               android:id="@+id/itemTitle" android:layout_toRightOf="@+id/itemImage" android:textSize="20dip">    
         </TextView>    
         <TextView     
               android:text="TextView02"     
               android:layout_height="wrap_content"     
               android:layout_width="fill_parent"     
               android:id="@+id/itemText" android:layout_toRightOf="@+id/itemImage" android:layout_below="@+id/itemTitle">    
         </TextView>    
</RelativeLayout>   

```
```

public class testListView extends Activity {  
    ListView listView;  
    String[] titles={"标题1","标题2","标题3","标题4"};  
    String[] texts={"文本内容A","文本内容B","文本内容C","文本内容D"};  
    int[] resIds={R.drawable.icon,R.drawable.icon,R.drawable.icon,R.drawable.icon};  
      
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        this.setTitle("BaseAdapter for ListView");  
        listView=(ListView)this.findViewById(R.id.listView1);  
        listView.setAdapter(new ListViewAdapter(titles,texts,resIds));  
    }  
  
    public class ListViewAdapter extends BaseAdapter {  
        View[] itemViews;  
  
        public ListViewAdapter(String[] itemTitles, String[] itemTexts,  
                int[] itemImageRes) {  
            itemViews = new View[itemTitles.length];  
  
            for (int i = 0; i < itemViews.length; i++) {  
                itemViews[i] = makeItemView(itemTitles[i], itemTexts[i],  
                        itemImageRes[i]);  
            }  
        }  
  
        public int getCount() {  
            return itemViews.length;  
        }  
  
        public View getItem(int position) {  
            return itemViews[position];  
        }  
  
        public long getItemId(int position) {  
            return position;  
        }  
  
        private View makeItemView(String strTitle, String strText, int resId) {  
            LayoutInflater inflater = (LayoutInflater) testListView.this  
                    .getSystemService(Context.LAYOUT_INFLATER_SERVICE);  
  
            // 使用View的对象itemView与R.layout.item关联  
            View itemView = inflater.inflate(R.layout.item, null);  
  
            // 通过findViewById()方法实例R.layout.item内各组件  
            TextView title = (TextView) itemView.findViewById(R.id.itemTitle);  
            title.setText(strTitle);  
            TextView text = (TextView) itemView.findViewById(R.id.itemText);  
            text.setText(strText);  
            ImageView image = (ImageView) itemView.findViewById(R.id.itemImage);  
            image.setImageResource(resId);  
              
            return itemView;  
        }  
  
        public View getView(int position, View convertView, ViewGroup parent) {  
            if (convertView == null)  
                return itemViews[position];  
            return convertView;  
        }  
    }  
  
}  

```
![](/data/dokuwiki/android/pasted/20150608-175200.png)
列表视图是一个成列显示滚动项的视图组合。成列的项都通过Adapter被自动插入到列表中，其中，Adapter适配器可以从数组或者数据库查询中提取出内容并转化成列表视图中的项。
**使用一个CursorLoader是避免一个异步任务查询光标Cursor时阻塞程序主线程的标准途径**。当CursorLoader收到一个Cursor结果，LoaderCallbacks会收到一个对onLoadFinished()的回调，这时可以利用新的Cursor和列表视图更新Adapter并显示结果。
下面的例子是把ListView作为唯一默认布局元素的活动ListActivity。它完成向Contacts Provider查询姓名和电话号码清单的功能。
为了使用CursorLoader向列表动态加载数据，这个活动实现了LoaderCallbacks接口。
```

public class ListViewLoader extends ListActivity
        implements LoaderManager.LoaderCallbacks<Cursor> {
 
    // This is the Adapter being used to display the list's data
    SimpleCursorAdapter mAdapter;
 
    // These are the Contacts rows that we will retrieve
    static final String[] PROJECTION = new String[] {ContactsContract.Data._ID,
            ContactsContract.Data.DISPLAY_NAME};
 
    // This is the select criteria
    static final String SELECTION = "((" + 
            ContactsContract.Data.DISPLAY_NAME + " NOTNULL) AND (" +
            ContactsContract.Data.DISPLAY_NAME + " != '' ))";
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        // Create a progress bar to display while the list loads
        ProgressBar progressBar = new ProgressBar(this);
        progressBar.setLayoutParams(new LayoutParams(LayoutParams.WRAP_CONTENT,
                LayoutParams.WRAP_CONTENT, Gravity.CENTER));
        progressBar.setIndeterminate(true);
        getListView().setEmptyView(progressBar);
 
        // Must add the progress bar to the root of the layout
        ViewGroup root = (ViewGroup) findViewById(android.R.id.content);
        root.addView(progressBar);
 
        // For the cursor adapter, specify which columns go into which views
        String[] fromColumns = {ContactsContract.Data.DISPLAY_NAME};
        int[] toViews = {android.R.id.text1}; // The TextView in simple_list_item_1
 
        // Create an empty adapter we will use to display the loaded data.
        // We pass null for the cursor, then update it in onLoadFinished()
        mAdapter = new SimpleCursorAdapter(this, 
                android.R.layout.simple_list_item_1, null,
                fromColumns, toViews, 0);
        setListAdapter(mAdapter);
 
        // Prepare the loader.  Either re-connect with an existing one,
        // or start a new one.
        getLoaderManager().initLoader(0, null, this);
    }
 
    // Called when a new Loader needs to be created
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        // Now create and return a CursorLoader that will take care of
        // creating a Cursor for the data being displayed.
        return new CursorLoader(this, ContactsContract.Data.CONTENT_URI,
                PROJECTION, SELECTION, null, null);
    }
 
    // Called when a previously created loader has finished loading
    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        // Swap the new cursor in.  (The framework will take care of closing the
        // old cursor once we return.)
        mAdapter.swapCursor(data);
    }
 
    // Called when a previously created loader is reset, making the data unavailable
    public void onLoaderReset(Loader<Cursor> loader) {
        // This is called when the last Cursor provided to onLoadFinished()
        // above is about to be closed.  We need to make sure we are no
        // longer using it.
        mAdapter.swapCursor(null);
    }
 
    @Override 
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Do something when a list item is clicked
    }
}

```
注意：因为这个例子要向Contacts Provider请求查询数据，程序需要在制作清单文件中请求READ_CONTACTS权限：
<uses-permission android:name="android.permission.READ_CONTACTS" />