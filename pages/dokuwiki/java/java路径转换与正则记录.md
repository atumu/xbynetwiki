title: java路径转换与正则记录 

#  java路径转换与正则记录 
```

public class Test1 {
	static String str="E:\\program\\tomcat7\\webapps\\resource\\b266f7ed-5957-4b4e-b2a2-7f1502121420.png";
	public static void main(String[] args) {
		String s=str.replaceAll("\\\\", "/"); //注意，赋值保存返回值。否则str任是原来的那个。这种低级错误已经犯了很多次了。
		System.out.println(s);
	}
}
输出E:/program/tomcat7/webapps/resource/b266f7ed-5957-4b4e-b2a2-7f1502121420.png

```
```

public class PathUtil {
	public static void pathChange(String path){
		if(path.indexOf("/")>=0){
			path=path.replaceAll("/", "\\\\");
		}else if(path.indexOf("\\")>=0){
			path.replaceAll("\\\\", "/");
		}
		System.out.println(path);
	}
	public static void main(String[] args) {
		pathChange("/sadadas/aaa");
          pathChange("\\sadadas\\aaa");
         System.out.println("/sadadas/aaa".replaceAll("/","\\\\\\\\"));
	}
}
输出:
\sadadas\aaa
/sadadas/aaa 
\\sadadas\\aaa

```
