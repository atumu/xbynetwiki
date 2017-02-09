title: root权限申请 

#  root权限申请 

概述
app在root过的设备当中申请root权限。

实现
```

public static boolean isRoot()  
    {  
        Process process = null;  
        DataOutputStream os = null;  
        try  
        {  
            process = Runtime.getRuntime().exec("su");  
            os = new DataOutputStream(process.getOutputStream());  
            os.writeBytes("exit\n");  
            os.flush();  
            int exitValue = process.waitFor();  
            if (exitValue == 0)  
            {  
                return true;  
            } else  
            {  
                return false;  
            }  
        } catch (Exception e)  
        {  
            Log.d(TAG, "Unexpected error - Here is what I know: "  
                    + e.getMessage());  
            return false;  
        } finally  
        {  
            try  
            {  
                if (os != null)  
                {  
                    os.close();  
                }  
                process.destroy();  
            } catch (Exception e)  
            {  
                e.printStackTrace();  
            }  
        }  
    }  

```