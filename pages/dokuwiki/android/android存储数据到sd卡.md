title: android存储数据到sd卡 

#  Android存储数据到SD卡 
##  权限 

在程序中访问SDCard，**你需要申请访问SDCard的权限**。
在AndroidManifest.xml中加入访问SDCard的权限如下:
''<!-- 在SDCard中创建与删除文件权限 -->
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<!-- 往SDCard写入数据权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>''
##  检测sd卡状态 

当用户已经安装PC存储或者移除了提供外部存储的SD卡，外部存储则不能使用，**因此需要在访问前验证是否可用**。调用 ` Environment.getExternalStorageState() `可以来查询外部存储状态。如果返回状态等同于` MEDIA_MOUNTED `，那就可以读取和写入文件。例如下列方法对确定可用存储很有帮助：
```

/* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

```
##  写入sd卡 

尽管外部存储可以由用户和其他应用程序进行修改，有两类文件可以在外部存储：
  * 公共文件:其他应用程序和用户可以自由访问该文件。**卸载应用程序时，用户仍然可以使用这些文件**。例如从应用程序或者其他下载文件获得的照片。
  * 私密文件:` /sdcard/Android/data<pkg_name> `**卸载应用程序时，属于该应用程序的私密文件应该删除**。尽管从技术角度由于外部存储，用户和其他应用程序可以访问这些文件。**用户卸载应用程序时，系统会删除应用程序外部私密目录中的所有文件。**
例如应用程序或者临时媒体文件下载的其他资源。如果想要在外部存储**公共文件**，请使用` getExternalStoragePublicDirectory() ` 来获取外部存储相应目录的文件。这一方法指定了要保存的文件类型，这样就可以和其他公共文件逻辑相连接，比如**DIRECTORY_MUSIC 或 DIRECTORY_PICTURES**。例如：
```

public File getAlbumStorageDir(String albumName) {
    // Get the directory for the user's public pictures directory. 
    File file = new File(Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}

```
如果想保存应用程序**私有文件**，可以调用` getExternalFilesDir() `来获取相应的目录。这种方式创建的每个目录都会添加到包含所有应用程序外部存储文件的父类目录，**用户卸载应用程序时系统会删除这些文件。**
```

public File getAlbumStorageDir(Context context, String albumName) {
    // Get the directory for the app's private pictures directory. 
    File file = new File(context.getExternalFilesDir(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}

```
##  获取SD根目录 

` Environment.getExternalStorageDirectory(); `
```

public String getStorageDir(){  
         if(!(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED))){  
             return "";  
         }  
         File dirFile=Environment.getExternalStorageDirectory();  
         Log.d(TAG, dirFile.getAbsolutePath());  
         return dirFile.getAbsolutePath();  
    } 

```

##  查询可用空间 
如果提前知道要保存的数据量，不需要调用 getFreeSpace() 和 getTotalSpace()就可以找出是否有足够的可用空间。这些方法分别提供了当前可用空间和存储总体空间。这一信息也有助于避免过度填充存储容量。但是系统并不能保证你可以写出与 getFreeSpace()表示相同多的字节数。如果返回的数字比想保存的数据超出几MB大小，或者文件系统小于90%总量，那就可能安全运行。否则就不应该写入存储。
```

public static long getAvailableSize(String path){  
    try{  
        File base = new File(path);  
    StatFs stat = new StatFs(base.getPath());  
    long nAvailableCount = stat.getBlockSize() * ((long) stat.getAvailableBlocks());  
    return nAvailableCount;  
    }catch(Exception e){  
    e.printStackTrace();  
    }  
    return 0;  
    } 

``` 
返回bytes单位的大小。
##  删除文件 
不再需要的文件就应该删除。删除文件最直接的方法是打开的文件引用自身调用delete()。
` myFile.delete(); `
