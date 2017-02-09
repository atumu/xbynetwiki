title: httpclient4.5学习之解决https证书问题 

#  HttpClient4.5学习之解决HTTPS证书问题 
在项目中正好需要访问https协议的接口，而对应的服务器没有购买商业CA颁发的正式受信证书，只是做了个自签名（联想一下12306网站购票时提示的那个警告信息），默认情况下通过HttpClient访问会抛出异常.
接下来谈下如何使用新的HttpClient来访问自签名https接口。
实现访问自签名https的要点就是建立一个自定义的SSLContext对象，该对象要有可以存储信任密钥的容器KeyStore，还要有判断当前连接是否受信任的策略，以及在SSL连接工厂中取消对所有主机名的验证。
1、首先建立一个信任任何密钥的策略。代码很简单，不去考虑证书链和授权类型，均认为是受信任的：
```

class AnyTrustStrategy implements TrustStrategy{  
      
    @Override  
    public boolean isTrusted(X509Certificate[] chain, String authType) throws CertificateException {  
        return true;  
    }  
      
}  

```
HttpClient既能处理常规http协议，又能支持https，**根源在于在连接管理器中注册了不同的连接创建工厂**。当访问url的schema为http时，调用明文连接套节工厂来建立连接；当访问url的schema为https时，调用SSL连接套接字工厂来建立连接。对于http的连接我们不做修改，只针对使用SSL的https连接来进行自定义：
```

RegistryBuilder<ConnectionSocketFactory> registryBuilder = RegistryBuilder.<ConnectionSocketFactory>create();  
ConnectionSocketFactory plainSF = new PlainConnectionSocketFactory();  
registryBuilder.register("http", plainSF);  
//指定信任密钥存储对象和连接套接字工厂  
try {  
    KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());  
    SSLContext sslContext = SSLContexts.custom().useTLS().loadTrustMaterial(trustStore, new AnyTrustStrategy()).build();  
    LayeredConnectionSocketFactory sslSF = new SSLConnectionSocketFactory(sslContext, SSLConnectionSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);  
    registryBuilder.register("https", sslSF);  
} catch (KeyStoreException e) {  
    throw new RuntimeException(e);  
} catch (KeyManagementException e) {  
    throw new RuntimeException(e);  
} catch (NoSuchAlgorithmException e) {  
    throw new RuntimeException(e);  
}  
Registry<ConnectionSocketFactory> registry = registryBuilder.build();  

```
在上述代码中可以看到，首先建立了一个密钥存储容器，随后让SSLContext开启TLS，并将密钥存储容器和信任任何主机的策略加载到该上下文中。构造SSL连接工厂时，将自定义的上下文和允许任何主机名通过校验的指令一并传入。最后将这样一个自定义的SSL连接工厂注册到https协议上。
前期准备已经完成，接下来我们要获得HttpClient对象：
```

//设置连接参数
connConfig = ConnectionConfig.custom().setCharset(Charset.forName(defaultEncoding)).build();
socketConfig = SocketConfig.custom().setSoTimeout(100000).build();
//设置连接管理器  
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager(registry);  
connManager.setDefaultConnectionConfig(connConfig);  
connManager.setDefaultSocketConfig(socketConfig);  
//构建客户端  
HttpClient client= HttpClientBuilder.create().setConnectionManager(connManager).build();  

```
为了让我们的HttpClient具有多线程处理的能力，连接管理器选用了PoolingHttpClientConnectionManager，将协议注册信息传入连接管理器，最后再次利用构造器的模式创建出我们需要的HttpClient。随后的GET/POST请求发起方法http和https之间没有差异。
为了验证我们的代码是否成功，可以做下JUnit单元测试：
```

@Test  
public void doTest() throws ClientProtocolException, URISyntaxException, IOException{  
    HttpUtil util = HttpUtil.getInstance();  
    InputStream in = util.doGet("https://kyfw.12306.cn/otn/leftTicket/init");  
    String retVal = HttpUtil.readStream(in, HttpUtil.defaultEncoding);  
    System.out.println(retVal);  
} 

``` 
参考：http://blog.csdn.net/chaijunkun/article/details/40145685