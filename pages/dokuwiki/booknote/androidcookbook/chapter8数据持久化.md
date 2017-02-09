title: chapter8数据持久化 

#  数据持久性 
Created Sunday 15 February 2015

##  读取应用自带而非文件系统中的文件 
你需要访问存储在res/raw目录中（而非文件系统/data,/sdcard/,/mnt）的一个文件
解决方案：使用getResources()和openRawResource()方法打开样例文件。
InputStreamReader is=new InputStreamReader(this.getResources().openRawResource(R.raw.samplefile);
BufferedReader bf=new BufferedReader(is);
....

##  列出目录 
java.io.File的list()或listFiles().以及用FilenameFilter进行过滤。

##  获取关于SD卡的总空间和可用空间的信息 
解决方案：android.os包中的StatFs和Environment类求得SD卡上的总空间和可用空间。
StatFs statFs=new StatFs(Environment.getExternalStorageDirectory().getPath());
double bytesTotal=(long)statFs.getBlockSize()*(long)statFs.getBlockCount();
double megTotal=bytesTotal/1048576;
double canuse=(long)statFs.getBlockSize()*(long)statFs.getAvailableBlocks();
如果希望显示有两位小叔的结果，可以用java.text.DecimalFormat对象
DecimalFormat twoDecimalForm=new DecimalFormat("#.##");

##  花费最小的精力提供用户首选项设置活动 
解决：子类化PreferenceActivity,在onCreate()中加载XML PreferenceScreen
讨论：SharedPreferences
sharedPref=PreferenceManager.getDefaultSharedPreferences(this);
可以有多个PreferenceScreen为根元素的xml布局元素。
在onCreate()中调用addPreferencesFromResource(R.layout.prefs1);
```

<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
	<ListPreference
		android:key="listChoice"
		android:title="List Choice"
		android:entries="@array/choices"
		android:entryValues="@array/choices"
		/>
	<PreferenceCategory 
		android:title="Personal">
		<EditTextPreference
			android:key="nameChoice"
			android:title="Name"
			android:hint="Name"
		/>
		<CheckBoxPreference
			android:key="booleanChoice"
			android:title="Binary Choice"
		/>	
	</PreferenceCategory>
</PreferenceScreen>

```
##  检查默认共享首选项的一致性 
PreferenceActivity的onSharedPreferenceChanged();或者OnSharedPreferenceChangeListener
不过要让SharedPreferences注册registerOnSharedPreferenceChangeListener();

##  执行高级文本搜索 
使用SQLite全文搜索3（SQLite Full Text Search 3 , FTS3）虚拟表和SQLite中的匹配数据层，存储和搜索文本数据。

####  一般情况下，最好使用内部数据库版本以控制升级。 

##  在Android应用程序中创建SQLite数据库 
SQLite数据库在Android中常规的使用方法就是扩展SQLiteOpenHelper类

##  在SQLite数据库中插入数值 
使用insert()方法，传递一个ContentValues类型的对象。
ContentValues提供与键值对类似的结构。

##  从现有的SQLite数据库加载数值 
解决方案：使用数据库的query()方法。并使用返回的Cursor对象遍历数据库并处理数据。

##  在SQLite中使用日期 
阐述使用SQLite的strftime()函数在SQLite时间戳格式和java.util.Date的转换
DDL: add_on TIMESTAMP NOT NULL DEFAULT **current_timestamp**
strftime('%s', add_on)*1000得到毫秒数。

##  使用JSONObject解析JSON 
JSONObjecct jo=new JSONObject();
try{
	jo.put("","");
	...
	
}catch(JSONException e){
}
...

##  使用DOM API解析XML文档 

##  使用XMLPullParser解析XML文档 

##  联系人添加和读 
