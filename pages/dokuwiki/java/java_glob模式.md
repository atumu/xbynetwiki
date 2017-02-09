title: java_glob模式 

#  glob模式语法与Filesystem类 
http://docs.oracle.com/javase/tutorial/essential/io/fileOps.html#glob
https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html#getPathMatcher-java.lang.String-

glob模式比正则表达式简单，其基本规则如下:
```

* 匹配0或多个字符
** 跨越目录匹配0或多个字符。如/opt/**,匹配/opt/html,/opt/html/www
? 匹配单个字符
{} 限定一个子模式集合，隐藏OR关系。如{java,php}.* 匹配所有以java.或php.开头的字符串。
[] 匹配一组字符串中的单个字符或者一组范围。如[0-9]匹配0到9之间的一个数字。[a-z,A-Z]
\ 转义符

```
  * {sun,moon,stars} matches "sun", "moon", or "stars".
  * {temp*,tmp*} matches all strings beginning with "temp" or "tmp".
  * [aeiou] matches any lowercase vowel.
  * [0-9] matches any digit.
  * [A-Z] matches any uppercase letter.
  * [a-z,A-Z] matches any uppercase or lowercase letter.
  * *.html – Matches all strings that end in .html
  * ??? – Matches all strings with exactly three letters or digits
  * *[0-9]* – Matches all strings containing a numeric value
  * *.{htm,html,pdf} – Matches any string ending with .htm, .html or .pdf
  * a?*.java – Matches any string beginning with a, followed by at least one letter or digit, and ending with .java
  * {foo*,*[0-9]*} – Matches any string beginning with foo or any string containing a numeric value


` FileSystem `是Java7引入的一个新类。其中有如下方法:
` public abstract PathMatcher getPathMatcher(String syntaxAndPattern) `
  * 参数形式:syntax:pattern
  * syntax的值支持"glob"、"regex"
  * pattern的值是对应syntax的模式字符串。
下面是一个Java程序，使用了Glob模式来搜索指定的目录及其子目录。
```

import java.io.IOException;
import java.nio.file.FileSystems;
import java.nio.file.FileVisitResult;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.PathMatcher;
import java.nio.file.Paths;
import java.nio.file.SimpleFileVisitor;
import java.nio.file.attribute.BasicFileAttributes;

public class FileGlobNIO {

    public static void main(String args[]) throws IOException {
        String glob = "glob:**/*.zip";
        String path = "D:/";
        match(glob, path);
    }

    public static void match(String glob, String location) throws IOException {

        final PathMatcher pathMatcher = FileSystems.getDefault().getPathMatcher(
                glob);

        Files.walkFileTree(Paths.get(location), new SimpleFileVisitor<Path>() {

            @Override
            public FileVisitResult visitFile(Path path,
                    BasicFileAttributes attrs) throws IOException {
                if (pathMatcher.matches(path)) {
                    System.out.println(path);
                }
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc)
                    throws IOException {
                return FileVisitResult.CONTINUE;
            }
        });
    }

}

```
程序输出如下：
D:\AndroidLocation.zip
D:\Eclipse\7dec2014\eclipse-jee-kepler-R-win32-x86_64\workspace\.metadata\.mylyn\.tasks.xml.zip
D:\Eclipse\7dec2014\eclipse-jee-kepler-R-win32-x86_64\workspace\.metadata\.mylyn\repositories.xml.zip
D:\Eclipse\7dec2014\eclipse-jee-kepler-R-win32-x86_64\workspace\.metadata\.mylyn\tasks.xml.zip
D:\mysql-workbench-community-6.2.5-winx64-noinstall.zip
D:\workspace\Android\AndroidChatBubbles-master.zip
D:\workspace\Android\Google Chat\XMPPChatDemo.zip
D:\workspace\Android\Update-Android-UI-from-a-Service-master.zip
D:\workspace\Android Chat\AndroidDialog.zip
D:\workspace\Android Wear\AndroidWearPreview.zip


##  支持glob模式排除过滤的目录迭代 
```

	private List<String> getFileList(File dir,String[] exclusionGlobs){
		List<String> filePathList=new ArrayList<>();
		iteratorDir(filePathList, dir, exclusionGlobs);
		return filePathList;
	}
	private void iteratorDir(List<String> filePathList,File dir,String[] exclusionGlobs){
		String[] files=dir.list();
		for(String path:files){
			File f=new File(path);
			if(f.isDirectory()) {
				iteratorDir(filePathList, f, exclusionGlobs);
			}else{
				if(exclusionGlobs!=null){
					for(String exp:exclusionGlobs){
						String glob="glob:"+exp;
						PathMatcher pm=FileSystems.getDefault().getPathMatcher(glob);
						if(!pm.matches(Paths.get(path))){
							filePathList.add(path);
							break;
						}
					}
				}else{
					filePathList.add(path);
				}
				
			}
		}
	}

```
参考:http://blog.csdn.net/chszs/article/details/46482571