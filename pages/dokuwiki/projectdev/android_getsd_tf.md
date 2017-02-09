title: android_getsd_tf 

#  Android获取机身存储、内置SD卡与外置TF卡路径 

#  获取机身存储路径(可以通过openFileInput,openFileOutput进行操作) 

String path=Environment.getDataDirectory().getAbsolutePath();返回/data  
#  获取内置SD卡路径: 
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
返回/storage/emulated/o

#  获取外置TF卡路径： 

思路：通过linux中的mount命令。
```

public String getTFDir(){  
        String path="";  
        try {  
            InputStream ins=Runtime.getRuntime().exec("mount").getInputStream();  
            BufferedReader reader=new BufferedReader(new InputStreamReader(ins));  
            String line="";  
            while((line=reader.readLine())!=null){  
                if(line.contains("sdcard")){  
                    if(line.contains("vfat")||line.contains("fuse")){  
                        String split[]=line.split(" ");  
                        path=split[1];  
                        Log.d(TAG,path);  
  
                    }  
                }  
            }  
          
            reader.close();  
            ins.close();  
              
        } catch (IOException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }finally{  
              
        }  
        return path;  
    }  

```
返回/storage/sdcard1这就是我们想要的路径。
#  获取可用空间 
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
