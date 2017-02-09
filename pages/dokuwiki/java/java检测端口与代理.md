title: java检测端口与代理 

#  Java检测端口、代理等检测 
##  检测端口是否打开 
```

package demo;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;

public class PortTest {
    public static void main(String[] args)  {
        Socket s=new Socket();
        InetSocketAddress addr=new InetSocketAddress("wiki.xby1993.net",80);
        try {
            s.connect(addr,1000);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(s!=null) {
                try {
                    s.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}


```
##  检测HTTP代理是否有效 
```

package demo;

import java.io.IOException;
import java.net.*;


public class ProxyTest {
    public static void main(String[] args)  {
//        Socket s=new Socket();
        InetSocketAddress addr=new InetSocketAddress("127.0.0.1",1080);
        Proxy proxy=new Proxy(Proxy.Type.HTTP,addr);
        HttpURLConnection conn=null;
        try {
            URL url=new URL("https://www.google.com.hk");
            conn= (HttpURLConnection) url.openConnection(proxy);
            conn.setConnectTimeout(30000);
            conn.connect();
            System.out.println(addr.getHostName()+":success");
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally{
            conn.disconnect();
        }
    }
}


```