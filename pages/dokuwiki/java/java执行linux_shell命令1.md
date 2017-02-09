title: java执行linux_shell命令1 

#  java执行linux shell命令并切换工作目录 
##  Java执行shell命令 
使用到Process和Runtime两个类，返回值通过Process类的getInputStream()方法获取
```

public class ReadCmdLine {  
    public static void main(String args[]) {  
        Process process = null;  
        List<String> processList = new ArrayList<String>();  
        try {  
            process = Runtime.getRuntime().exec("ps -aux");  
            BufferedReader input = new BufferedReader(new InputStreamReader(process.getInputStream()));  
            String line = "";  
            while ((line = input.readLine()) != null) {  
                processList.add(line);  
            }  
            input.close();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
  
        for (String line : processList) {  
            System.out.println(line);  
        }  
    }  
} 

``` 
调用shell脚本，判断是否正常执行，**如果正常结束，Process的waitFor()方法返回0**
```

public static void callShell(String shellString) {  
    try {  
        Process process = Runtime.getRuntime().exec(shellString);  
        int exitValue = process.waitFor();  
        if (0 != exitValue) {  
            log.error("call shell failed. error code is :" + exitValue);  
        }  
    } catch (Throwable e) {  
        log.error("call shell failed. " + e);  
    }  
}  

```

----
**java.lang.Runtime:**
Process	exec(String command)
Executes the specified string command in a separate process.
Process	exec(String[] cmdarray)
Executes the specified command and arguments in a separate process.
Process	exec(String[] cmdarray, String[] envp)
Executes the specified command and arguments in a separate process with the specified environment.
Process	exec(String[] cmdarray, String[] envp, File dir)
Executes the specified command and arguments in a separate process with the specified environment and working directory.
Process	exec(String command, String[] envp)
Executes the specified string command in a separate process with the specified environment.
Process	exec(String command, String[] envp, File dir)
Executes the specified string command in a separate process with the specified environment and working directory.
void	exit(int status)
Terminates the currently running Java virtual machine by initiating its shutdown sequence.

**java.lang.Process:**
abstract void	destroy()
Kills the subprocess.
abstract int	exitValue()
Returns the exit value for the subprocess.
abstract InputStream	getErrorStream()
Returns the input stream connected to the error output of the subprocess.
abstract InputStream	getInputStream()
Returns the input stream connected to the normal output of the subprocess.
abstract OutputStream	getOutputStream()
Returns the output stream connected to the normal input of the subprocess.
abstract int	waitFor()
Causes the current thread to wait, if necessary, until the process represented by this Process object has terminated.
##  执行shell并切换工作目录 
以下代码执行的shell命令类似如下:
```

cd /home/test/abc && /usr/bin/rsync -artuz -R --delete /home/test/abc  --port=22210 testrsync@10.20.20.181::share2 --password-file=/etc/passwd.txt

```
由于需要切换工作目录cd /home/test/abc

` 错误的做法: `
Process process=Runtime.getRuntime().exec("cd /home/test/abc && /usr/bin/rsync -artuz -R --delete /home/test/abc  --port=22210 testrsync@10.20.20.181::share2 --password-file=/etc/passwd.txt");

` 正确的做法: `
```

String[] cmd = { "/bin/sh", "-c", stb.toString()};
Process process=Runtime.getRuntime().exec(cmd);

```

----

```

public static void pushSite(SiteSyncInfo info){
		final StringBuilder stb=new StringBuilder("cd ");
		String sitePath=Paths.get(FileUtil.getCMSWebConfig(GlobalConstract.PUBLISH_PATH_KEY),info.getSitePath()).toString();
		stb.append(sitePath+" && ");
		String rsyncPath=FileUtil.getCMSWebConfig(GlobalConstract.RSYNC_PATH_KEY);
		stb.append(rsyncPath+" -artuz -R --delete ./ ");
		stb.append(" --port="+info.getRemotePort().trim());
		stb.append(" "+info.getRemoteUser().trim()+"@"+info.getRemoteAddr()+"::"+info.getSiteShareName());
		stb.append(" --password-file="+info.getPasswdFilePath()+" ");
//		#cd /home/test/abc && /usr/bin/rsync -artuz -R --delete /home/test/abc  --port=22210 testrsync@10.20.20.181::share2 --password-file=/etc/passwd.txt
		ThreadPoolUtil.getPushTaskPool().submit(new Runnable(){
			@Override
			public void run(){
				try {
					log.info("site force push shell cmd is {}",stb.toString());
					String[] cmd = { "/bin/sh", "-c", stb.toString()};
					Process process=Runtime.getRuntime().exec(cmd);
					int exitValue = process.waitFor();  
			        if (0 != exitValue) {  
			            log.error("call shell failed. error code is :" + exitValue);  
			        } 
				} catch (IOException | InterruptedException e) {
					e.printStackTrace();
					log.error("when running the site force push shell cmd {} ,there is exception happened",stb.toString());
					log.error(e);
				} 
			}
		});
	}

```
参考:
http://blog.csdn.net/arkblue/article/details/7897396
http://stackoverflow.com/questions/4884681/how-to-use-cd-command-using-java-runtime