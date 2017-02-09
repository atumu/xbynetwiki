title: chapter1000问题 

#  问题与解决 
Created Sunday 01 February 2015

问题1：android studio中SimpleTorch项目突然出现can't resolve symbol 'R'
解决：build-clean project或者重启android studio之后再clean。重复几次。或者更改compile sdk等等。
问题2：对资源如摄像头出现null引用问题
解决：AndroidManifest.xml中添加相关权限代码。
问题3：应用程序重启时FC:
解决：可能原因之一是相关硬件资源被占用而无法获取，。
问题4：没有也无法生成R.java文件
解决：今天导入一个项目时出现了gen目录下只有buildconfig.java文件而没有R.java。project clean以及删除gen目录完全没用。这种情况产生原因有很多。
找了半天，原因就是该项目的target sdk version与eclipse默认的不一致。右键项目properties在android选项中更改默认的Project build path即可。