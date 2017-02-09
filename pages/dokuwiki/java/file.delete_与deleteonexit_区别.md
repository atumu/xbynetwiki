title: file.delete_与deleteonexit_区别 

#  File.delete()与deleteOnExit()区别 
File.delete()与deleteOnExit()区别:
  * deleteOnExit()方法是` 虚拟机终止时才进行删除 `。 一般用于删除程序建立的临时文件，当程序关闭时便自动删除。
  * delete()方法就是普通的删除。一般用于删除普通文件。

由于没有事先搞清楚它们的关系，今日在做备份移动硬盘的工具时，` 使用File.deleteOnExit()方法发现日志提示删除了，但是并没有实际删除。一查，原来如此。改为File.delete()方法即可。 `

deleteOnExit()删除临时文件示例：
```

import java.io.File;
public class FileDemo {
   public static void main(String[] args) {
      File f = null;           
      try{
         // creates temporary file
         f = File.createTempFile("tmp", ".txt");          
         // prints absolute path
         System.out.println("File path: "+f.getAbsolutePath());          
         // deletes file when the virtual machine terminate
          f.deleteOnExit();          
         // creates temporary file
         f = File.createTempFile("tmp", null);          
         // prints absolute path
         System.out.print("File path: "+f.getAbsolutePath());          
         // deletes file when the virtual machine terminate
         f.deleteOnExit();         
      }catch(Exception e){
         // if any error occurs
         e.printStackTrace();
      }
   }
}

```