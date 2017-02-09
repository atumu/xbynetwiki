title: android官方文档学习之menu 

#  android官方文档学习之Menu 
  * 选项菜单和动作条
  * 上下文菜单和上下操作模式:上下文菜单是一种浮动的菜单，是在当用户在一个元件上执行**长按**动作时显示的。
  * 弹出窗口菜单
##  在XML中定义菜单 
` res/menu/ `目录下创建一个XML文件:
```

 <?xml version="1.0" encoding="utf-8"?>
 <menu xmlns:android="http://schemas.android.com/apk/res/android">    
 <item android:id="@+id/new_game"          
       android:icon="@drawable/ic_new_game"          
       android:title="@string/new_game"          
       android:showAsAction="ifRoom"/>    
 <item android:id="@+id/help"          
       android:icon="@drawable/ic_help"          
       android:title="@string/help" />
 </menu>

```
<item>元件支持多种属性，你可以用来定义一个项的样式和行为。菜单上的选项包含了以下属性：
` android:id `
菜单项唯一的的ID资源，当用户选中这个选项时允许应用通过这个ID来识别这个菜单项。
` android:icon `
索引一个图片资源作为该项的图标。
` android:title `
索引一个字符串作为该项的标题
` android:showAsAction `
载明该项作为一个行为项什么时候和怎样显示在动作条中。
```

 <?xml version="1.0" encoding="utf-8"?>
 <menu xmlns:android="http://schemas.android.com/apk/res/android">    
      <item android:id="@+id/file"          
            android:title="@string/file" >        
          <!-- "file" submenu -->        
          <menu>            
              <item android:id="@+id/create_new"                  
                    android:title="@string/create_new" />            
              <item android:id="@+id/open"                  
                    android:title="@string/open" />        
          </menu>    
      </item>
 </menu>

```
在你的应用中使用菜单，你可以使用` MenuInflater.inflate() `寻找需要的菜单资源文件
##  创建一个选项菜单 
```

 @Override
 public boolean onCreateOptionsMenu(Menu menu) {    
    MenuInflater inflater = getMenuInflater();    
    inflater.inflate(R.menu.game_menu, menu);    
    return true;
 }

```
##  单击事件处理 
```

 @Override
 public boolean onOptionsItemSelected(MenuItem item) {
    // Handle item selection
    switch (item.getItemId()) {
        case R.id.new_game:
            newGame();
            return true;
        case R.id.help:
            showHelp();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
 }

```
##  创建菜单组 
菜单组是一个分享某些特征的菜单项集合，使用菜单组，你可以：
使用setGroupVisible()显示或隐藏所有选项；
使用setGroupEnabled()让所有选项有效或无效；
使用setGroupCheckable()指定是否所有选项可选。
你可以在<group>元件里嵌套<item>元件来创建菜单组。
```

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
   <item android:id="@+id/menu_save"
         android:icon="@drawable/menu_save"
         android:title="@string/menu_save" />
   <group android:id="@+id/group_delete">
       <item android:id="@+id/menu_archive"
             android:title="@string/menu_archive" />
       <item android:id="@+id/menu_delete"
             android:title="@string/menu_delete" />
   </group>
</menu>

```
###  使用可选的菜单项 
![](/data/dokuwiki/android/pasted/20150608-163634.png)
你可以在` <item>元素使用android:checkable `属性来定义一个独立菜单项的可选行为，或者在` <group>元素里使用android:checkableBehavior属性来为整个组定义 `。例如，这个菜单组里的所有选项使用单选按钮且可选：
```

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
   <group android:checkableBehavior="single">
       <item android:id="@+id/red"
             android:title="@string/red" />
       <item android:id="@+id/blue"
             android:title="@string/blue" />
   </group>
</menu>

```
android:checkableBehavior属性可设定为：
single
在菜单组中只有一个选项能被选中（单选按钮）
all
所有选项都可被选（复选框）
none
没有选项可选
##  基于一个Intent添加菜单选项 

##  创建上下文菜单 

为了提供一个浮动的上下文菜单：
1、通过调用` resisterForContextMenu(View) `来注册上下文菜单相关的视，并在视图中通过它。
如果你的活动使用了ListView或者GridView且你想要每个选项都提供一个相同的上下文菜单，那么需要通过调用ListView或者GridView中的` registerForContextMenu() `为一个上下文菜单注册所有的选项。
2、在你的活动(Activity)或者片段（Fragment）实现` onCreateContextMenu() `的方法。(注意：` onCreateContextMenu()、onCreateOptionsMenu() `)
当注册时接收到一个长点击事件，那么系统将会调用你的onCreateContextMenu方法。这是你定义菜单项的地方，通常通过导入一个菜单资源。例如：
```

    @Override
    protectedvoid onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 显示列表
        simpleShowList();
        // 为所有列表项注册上下文菜单
	  this.registerForContextMenu(getListView());
    }
 @Override
 public void onCreateContextMenu(ContextMenu menu, View v,
                                ContextMenuInfo menuInfo) {
    super.onCreateContextMenu(menu, v, menuInfo);
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.context_menu, menu);
 }

```
3、实现` onContextItemSelected()。 `
当用户选择一个菜单项时，系统调用这个方法则你可以执行相应的动作。例如：
```

 @Override
 public boolean onContextItemSelected(MenuItem item) {
    AdapterContextMenuInfo info = (AdapterContextMenuInfo) item.getMenuInfo();
    switch (item.getItemId()) {
        case R.id.edit:
            editNote(info.id);
            return true;
        case R.id.delete:
            deleteNote(info.id);
            return true;
        default:
            return super.onContextItemSelected(item);
    }
 }

```
##  创建弹出菜单 
弹出菜单是一个形式上的菜单标记在View上面。它出现在定位的窗口之下，如果那里有空间的话，或者在其上面。
```

public void showPopup(View v) {
    PopupMenu popup = new PopupMenu(this, v);
    MenuInflater inflater = popup.getMenuInflater();
    inflater.inflate(R.menu.actions, popup.getMenu());
    popup.show();
}
 @Override
 public boolean onMenuItemClick(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.archive:
            archive(item);
            return true;
        case R.id.delete:
            delete(item);
            return true;
        default:
            return false;
    }
 }

```