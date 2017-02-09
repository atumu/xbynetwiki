title: getresourceasstream问题 

#  getSystemResourceAsStream与getResourceAsStream问题 
最近发现在Main函数中直接用
getSystemResourceAsStream()可以获取
而在webapp运行时得到的却是null。
仔细查看ClassLoader的源代码发现：
```

 public static URL getSystemResource(String name) {
        ClassLoader system = getSystemClassLoader();//直接调用了SystemClassLoader了
        if (system == null) {
            return getBootstrapResource(name);
        }
        return system.getResource(name);
    }

```
而我们webapp的类加载器都不是SystemClassLoader而是对应应用容器的类加载器。所以就无法加载到相应的资源返回null了。
而javase环境通过main函数一般都是由SystemClassLoader直接加载的。所以不存在此问题。
**webapp下的classloader为如tomcat中(WebappClassLoader):**
```

WebappClassLoader
  context: 
  delegate: false
  repositories:
    /WEB-INF/classes/
----------> Parent Classloader:
org.apache.catalina.loader.StandardClassLoader@28161b52

```
**而javase环境main函数中classloader为:**
sun.misc.Launcher$AppClassLoader@333c339f也就是我们通常所说的SystemClassLoader

建议平常尽量用class.getResourceAsStream()。` 不过它默认根目录是WEB-INF/classes.而不能读取WEB-INF/jar中的类 `
