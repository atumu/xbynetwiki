title: java7_io操作 

#  java7 io操作新便利 
本文参考：
http://blog.csdn.net/zhongweijian/article/details/8451347 系列文章
http://www.mpabo.com/2014/11/23/java-nio-file-pathjdk7%E4%B8%8Ejava-io-file%E6%AF%94%E8%BE%83/
http://stackoverflow.com/questions/6903335/java-7-path-vs-file
#  Files,Path,FileSystems 
java 7 新的NIO API新增了FileSystem和FileStore相关API，最主要的是Path.java类，主要是我们常见的文件的一些相关操作，这个Path是NIO IO操作的基础，我们可以定义一个文件路径，对路径的相关操作等，
**Path 类是jdk7新增加的特性之一，用来代替java.io.File类。**之所以新增这个类，是由于java.io.File类有很多缺陷：
1.java.io.File类里面很多方法失败时没有异常处理，或抛出异常
2.java.io.File.rename(File file)方法在不同平台中运行时可能会有问题,这个方法不能将一个文件从一个文件系统移到另一个文件系统，这个方法的操作也不是原子性的，如果参数指定的文件名已经存在这个方法也可能执行失败：
3. 读取文件属性相关
File类中读取文件属性都是一个方法返回一个属性值，而没有能够直接一次返回很多属性的方法，造成访问文件属性时效率的问题。但是对于jkd7新增的api中可以批量读取文件属性，而且可以访问到文件更详细的属性。
```

Path path= FileSystems.getDefault().getPath(first, more)  
Path path = Paths.get(System.getProperty("user.home"), "www",  
                    "pyweb.settings"); 
File path_to_file = path.toFile();  
Path file_to_path = path_to_file.toPath();  


``` 

##  最常用Files方法 
```

获取文件
Path copy_from = Paths.get("/tmp", "draw_template.txt");  
Path path = FileSystems.getDefault().getPath(  
                System.getProperty("user.home"), "www", "pyweb.settings");  
 // 删除文件  
boolean success = Files.deleteIfExists(newdir2);  
//拷贝文件：
Files.copy(copy_from, copy_to);  
//移动文件
 Files.move(movefrom, moveto, StandardCopyOption.REPLACE_EXISTING);  
// 创建新文件  
        Path newfile = FileSystems.getDefault().getPath(  
                "/tmp/SonyEricssonOpen.txt");  
        Set<PosixFilePermission> perms1 = PosixFilePermissions  
                .fromString("rw-------");  
        FileAttribute<Set<PosixFilePermission>> attr2 = PosixFilePermissions  
                .asFileAttribute(perms1);  
            Files.createFile(newfile, attr2);  


// 读写文件缓存流操作  
        String text = "\nVamos Rafa!";  
        try (BufferedWriter writer = Files.newBufferedWriter(newfile, charset,  
                StandardOpenOption.APPEND)) {  
            writer.write(text);  
        } catch (IOException e) {  
            System.err.println(e);  
        }  
        try (BufferedReader reader = Files.newBufferedReader(newfile, charset)) {  
            String line = null;  
            while ((line = reader.readLine()) != null) {  
                System.out.println(line);  
            }  
        } catch (IOException e) {  
            System.err.println(e);  
        }
// 创建多级目录,创建bbb目录，在bbb目录下再创建ccc目录等等  
        Path newdir2 = FileSystems.getDefault().getPath("/tmp/aaa",  
                "/bbb/ccc/ddd");  
        try {  
            Files.createDirectories(newdir2);  
        } catch (IOException e) {  
            System.err.println(e);  
        }  
 // 列举目录信息  
        Path newdir3 = FileSystems.getDefault().getPath("/tmp");  
        try (DirectoryStream<Path> ds = Files.newDirectoryStream(newdir3)) {  
            for (Path file : ds) {  
                System.out.println(file.getFileName());  
            }  
        } catch (IOException e) {  
            System.err.println(e);  
        }  
 // 通过正则表达式过滤  
        System.out.println("\nGlob pattern applied:");  
        try (DirectoryStream<Path> ds = Files.newDirectoryStream(newdir3,  
                "*.{png,jpg,bmp，ini}")) {  
            for (Path file : ds) {  
                System.out.println(file.getFileName());  
            }  
        } catch (IOException e) {  
            System.err.println(e);  
        }
  // 临时目录操作  
        String tmp_dir_prefix = "nio_";  
        try {  
            // passing null prefix  
            Path tmp_1 = Files.createTempDirectory(null);  
            System.out.println("TMP: " + tmp_1.toString());  
            // set a prefix  
            Path tmp_2 = Files.createTempDirectory(tmp_dir_prefix);  
            System.out.println("TMP: " + tmp_2.toString());  
  
            // 删除临时目录  
            Path basedir = FileSystems.getDefault().getPath("/tmp/aaa");  
            Path tmp_dir = Files.createTempDirectory(basedir, tmp_dir_prefix);  
            File asFile = tmp_dir.toFile();  
            asFile.deleteOnExit();  
  
        } catch (IOException e) {  
            System.err.println(e);  
        }  

```
##  其他 

```

Path path = FileSystems.getDefault().getPath(  
                System.getProperty("user.home"), "www", "pyweb.settings");  
 // 获取文件系统根目录  
Iterable<Path> dirs = FileSystems.getDefault().getRootDirectories();  
 // 检测文件访问权限  
        boolean is_readable = Files.isReadable(path);  
        boolean is_writable = Files.isWritable(path);  
        boolean is_executable = Files.isExecutable(path);  
// 检测文件是否指定同一个文件  
 boolean is_same_file_12 = Files.isSameFile(path_1, path_2);  
// 检测文件可见行  
 boolean is_hidden = Files.isHidden(path);

 // 创建新目录  
        Path newdir = FileSystems.getDefault().getPath("/tmp/aaa");  
        // try {  
        // Files.createDirectory(newdir);  
        // } catch (IOException e) {  
        // System.err.println(e);  
        // }  
        Set<PosixFilePermission> perms = PosixFilePermissions  
                .fromString("rwxr-x---");  
        FileAttribute<Set<PosixFilePermission>> attr = PosixFilePermissions  
                .asFileAttribute(perms);  
        try {  
            Files.createDirectory(newdir, attr);  
        } catch (IOException e) {  
            System.err.println(e);  
        }  
  
 // 写小文件  
        try {  
            byte[] rf_wiki_byte = "test".getBytes("UTF-8");  
            Files.write(newfile, rf_wiki_byte);  
        } catch (IOException e) {  
            System.err.println(e);  
        }  
  
        // 读小文件  
        try {  
            byte[] ballArray = Files.readAllBytes(newfile);  
            System.out.println(ballArray.toString());  
        } catch (IOException e) {  
            System.out.println(e);  
        }  
        Charset charset = Charset.forName("ISO-8859-1");  
        try {  
            List<String> lines = Files.readAllLines(newfile, charset);  
            for (String line : lines) {  
                System.out.println(line);  
            }  
        } catch (IOException e) {  
            System.out.println(e);  
        }  


```  

#  1.Path/Paths 
```

1. //查找目录下文件
2. Path dir = Paths.get("E:LearnJava7trunk");
3. try (DirectoryStream<Path> stream = Files.newDirectoryStr
eam(dir,
4. "*.properties")) {
5. for (Path entry : stream) {
6. System.out.println(entry.getFileName());
7. }
8. } catch (IOException e) {
9. System.out.println(e.getMessage());
10. }

1. Path listing = Paths.get("/usr/bin/zip");
2.
3. System.out.println("File Name [" + listing.getFileName() + "]"
);
4. System.out.println("Number of Name Elements in the Path ["
5. + listing.getNameCount() + "]");
6. System.out.println("Parent Path [" + listing.getParent() + "]");
7. System.out.println("Root of Path [" + listing.getRoot() + "]");
8. System.out.println("Subpath from Root, 2 elements deep ["
9. + listing.subpath(0, 2) + "]");

```

#  2.Files.walkFileTree 遍历目录 
```

Path startingDir = Paths.get("E:LearnJava7trunk");
3. //遍历目录
4. Files.walkFileTree(startingDir, new FindJavaVisitor());
5. }
6.
7. private static class FindJavaVisitor extends SimpleFil
eVisitor<Path> {
8.
9. @Override
10. public FileVisitResult visitFile(Path file, BasicFileAttribute
s attrs) {
11.
12. if (file.toString().endsWith(".java")) {
13. System.out.println(file.getFileName());
14. }
15. return FileVisitResult.CONTINUE;
16. }


```
    
#  3.Files 
```

Path zip = Paths.get("E:javatest");
3. //Path zip = Paths.get("E:java测试.rar");
4. System.out.println(zip.toAbsolutePath().toString());
5. System.out.println(Files.getLastModifiedTime(zip));
6. System.out.println(Files.size(zip));
7. System.out.println(Files.isSymbolicLink(zip));
8. System.out.println(Files.isDirectory(zip));
9. System.out.println(Files.readAttributes(zip, "*"));

if(Files.exists(old)){
3. Files.copy(old, target, StandardCopyOption.REPLACE_
EXISTING);
4. Files.move(old, target2,StandardCopyOption.REPLACE
_EXISTING);
  
  if(Files.notExists(target)){
17. Files.createFile(target);
18. }
19.

```
  
#  4.读文件 
```

try(BufferedWriter writer =
16. Files.newBufferedWriter(target, StandardCharsets.UTF
_8, StandardOpenOption.WRITE)){
17. writer.write("hello");
18. } catch (IOException e) {
19.
20. e.printStackTrace();
21. }

try {
25. @SuppressWarnings("unused")
26. List<String> lines = Files.readAllLines(old, StandardChars
ets.UTF_8);
27. @SuppressWarnings("unused")
28. byte[] bytes = Files.readAllBytes(old);
29. } catch (IOException e) {
31. e.printStackTrace();
32. }


```

#  5.监听目录变化 
` WatchService ` 是线程安全的，跟踪文件事件的服务，一般是用独立线程启动跟踪  

```

   //WatchService 是线程安全的，跟踪文件事件的服务，一般是用独立线程启动跟踪  
    public void watchRNDir(Path path) throws IOException, InterruptedException {  
        try (WatchService watchService = FileSystems.getDefault().newWatchService()) {  
            //给path路径加上文件观察服务  
            path.register(watchService, StandardWatchEventKinds.ENTRY_CREATE,  
                    StandardWatchEventKinds.ENTRY_MODIFY,  
                    StandardWatchEventKinds.ENTRY_DELETE);  
            // start an infinite loop  
            while (true) {  
                // retrieve and remove the next watch key  
                final WatchKey key = watchService.take();  
                // get list of pending events for the watch key  
                for (WatchEvent<?> watchEvent : key.pollEvents()) {  
                    // get the kind of event (create, modify, delete)  
                    final Kind<?> kind = watchEvent.kind();  
                    // handle OVERFLOW event  
                    if (kind == StandardWatchEventKinds.OVERFLOW) {  
                        continue;  
                    }  
                    //创建事件  
                    if(kind == StandardWatchEventKinds.ENTRY_CREATE){  
                          
                    }  
                    //修改事件  
                    if(kind == StandardWatchEventKinds.ENTRY_MODIFY){  
                          
                    }  
                    //删除事件  
                    if(kind == StandardWatchEventKinds.ENTRY_DELETE){  
                          
                    }  
                    // get the filename for the event  
                    final WatchEvent<Path> watchEventPath = (WatchEvent<Path>) watchEvent;  
                  获取事件所属的文件或目录名
                    final Path filename = watchEventPath.context();  
                    // print it out  
                    System.out.println(kind + " -> " + filename);  
  
                }  
                // reset the keyf  
              记得重置key并获取其返回值用于判断key是否还有效，否则则可以判断该目录已被删除处于无效状态
                boolean valid = key.reset();  
                // exit loop if the key is not valid (if the directory was  
                // deleted, for  
                if (!valid) {  
                    break;  
                }  
            }  
        }  
    }  

```
  
  
