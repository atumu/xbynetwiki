title: chapter4内容提供者 


#  内容提供者 
Created Sunday 08 February 2015

Content Provider是Android较为精巧的想法之一，它允许完全不相关的应用程序共享数据，这些数据通常保存在SQLite数据库中。
Android Contact provider（联系人提供者）是广泛使用的内容提供者之一。
Android AIDL（接口定义语言）提供了分层远程过程机制。

##  从内容提供者获取数据 
例如从Contacts中获取数据
Uri uri=ContactsContract.Contacts.CONTENT__URI;
Intent intent=new Intent(Intent.ACTION_PICK,uri);
startActivityForResult(intent,requestCode);

onActivityResult部分代码：
	if(requestCode==。。。&& resultCode=Activity.RESULT_OK){
		Uri resultUri=data.getData();
		Cursor cursor=getContentResolver().query(resultUri,null,null,null,null);
		...
	}

##  编写内容提供者 
在Manifest中声明 
<provider android:authorities="...."
		android:name="..."/>
继承ContentProvider

##  编写Android远程服务 
基于AIDL与IPC(Inter-process Communication)

具体有待研读


