title: accesscontroller.doprivileged 

#  AccessController.doPrivileged特权使用 
AccessController 类用于与**访问控制相关**的操作和决定。
更确切地说，AccessController 类用于以下三个目的：
  * 基于当前生效的安全策略决定是允许还是拒绝对关键系统资源的访问，
  * 将代码标记为享有“**特权(privilege)**”，从而影响后续访问决定，以及
  * 获取当前调用上下文的“快照”，这样即可相对于已保存的上下文作出其他上下文的访问控制决定。

**checkPermission 方法确定应该批准还是拒绝由指定权限所指示的访问请求**。示例调用如下所示。在此例中，checkPermission 将确定是否批准对 "/temp" 目录中名为 "testFile" 的文件的“读”访问。
```

FilePermission perm = new FilePermission("/temp/testFile", "read");
AccessController.checkPermission(perm);

```
如果允许请求的访问，则 checkPermission 正常返回。如果拒绝，则抛出 AccessControlException。如果请求的权限类型不正确或包含无效值，则也会抛出 AccessControlException。
可以通过doPrivileged方法将调用方标记为享有“特权”（请参阅 doPrivileged 及下文）。在做访问控制决定时，如果遇到通过调用不带上下文参数（请参见下文以获取关于上下文参数的信息）的 **doPrivileged 标记为“特权”的调用方，则 checkPermission 方法将停止检查。** 
但是如果该调用方的域具有指定的权限，则不进行进一步检查并且 checkPermission 正常返回，指示允许所请求的访问。如果该域不具有指定的权限，则通常抛出异常。
一、"特权"特性的正常使用如下所示：

1、如果你不需要从"特权"块内返回一个值，按下列代码去做：
```

somemethod() {
      ...normal code here...
      AccessController.doPrivileged(new PrivilegedAction() {
             public Object run() {
                    // privileged code goes here, for example:
                    System.loadLibrary("awt");
                    return null; // nothing to return
            }
      });

      ...normal code here...

}

``` 
PrivilegedAction是一个接口，它带有一个被称为run的方法，这个方法返回一个Object。上述例子显示了一个用来实现那个接口的匿名内类的创建，并提供了一个run方法的具体实现。
当做一个doPrivileged调用时，一个PrivilegedAction实现的实例被传递给它。doPrivileged方法在使特权生效后，从PrivilegedAction实现中调用run方法，并返回run方法的返回值以作为doPrivileged的返回值，这一点在本例中被忽略。
2、如果你需要返回一个值，你可按如下方法去做：
```

somemethod() {

        ...normal code here...

        String user = (String) AccessController.doPrivileged(new PrivilegedAction() {
                  public Object run() {
                         return System.getProperty("user.name");
                 }
        });

        ...normal code here...

}

```
3、如果用你的run方法执行的动作可能扔出一个"检查"的异常（包括在一个方法的throws子句列表中），则你需要使用PrivilegedExceptionAction接口，而不是使用PrivilegedAction接口：

```

somemethod() throws FileNotFoundException {

        ...normal code here...

        try {
               FileInputStream fis = (FileInputStream)
               AccessController.doPrivileged(new PrivilegedExceptionAction() {
                     public Object run() throws FileNotFoundException {
                            return new FileInputStream("someFile");
                     }
               });
        } catch (PrivilegedActionException e) {
                 // e.getException() should be an instance of
                 // FileNotFoundException,
                 // as only "checked" exceptions will be "wrapped" in a
                 // PrivilegedActionException.
                throw (FileNotFoundException) e.getException();
        }

       ...normal code here...

}

```
有关被授予特权的一些重要事项：
  * 首先，这个概念仅存在于一个单独线程内。一旦特权代码完成了任务，特权将被保证清除或作废。
  * 第二，在这个例子中，**在run方法中的代码体被授予了特权**。然而，如果它调用无特权的不可信代码，则那个代码将不会获得任何特权；只有在特权代码具有许可并且在直到checkPermission调用的调用链中的所有随后的调用者也具有许可时, 一个许可才能被准予。

二、使用事例：
```

	@SuppressWarnings("unchecked")
	private static void unmap(final MappedByteBuffer byteBuffer){
		if(byteBuffer==null) return;
		AccessController.doPrivileged(new PrivilegedAction() {  
			  public Object run() {  
			    try {  
			      Method getCleanerMethod = byteBuffer.getClass().getMethod("cleaner", new Class[0]);  
			      getCleanerMethod.setAccessible(true);  
			      sun.misc.Cleaner cleaner = (sun.misc.Cleaner)   
			      getCleanerMethod.invoke(byteBuffer, new Object[0]);  
			      cleaner.clean();  
			      log.info("MappedByteBuffer释放完毕");
			    } catch (Exception e) {  
			      e.printStackTrace();
			      log.warn("内存释放异常{}",e);
			    }  
			    return null;  
			  }  
			});
	}

```
参考http://blog.csdn.net/gtuu0123/article/details/4512247
http://blog.csdn.net/teamlet/article/details/1809165