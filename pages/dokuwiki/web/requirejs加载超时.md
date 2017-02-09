title: requirejs加载超时 

#  requireJS加载超时处理 
出现这个问题有三种可能情形：
1、资源找不到，看看是否报404错误。
2、由于网络原因导致，网速过慢。可以修改配置参数waitSeconds (默认为7秒，可以设置为0表示永远不超时，或者其他正数)，
```

 require.config({
    baseUrl: "/another/path",
    paths: {
      "some": "some/v1.0"
    },
    waitSeconds: 15
  });

```
**3、由于在config配置时，对同一资源配置了多个可访问的别名**。**然后加载时不同地方使用了不同的别名导致加载超时**。如:
```

 require.config({
    baseUrl: "/another/path",
    paths: {
      "some": "some/v1.0",
      "someOther":"some/v1.0",
      "someDir":"some"
    },
    waitSeconds: 15
  });

```
然后加载时不同地方使用了不同的别名导致加载超时。如：
require(['some'],function(some){....}) 第一次加载some/v1.0出现的地方。加载正常。` 以后每次加载都应该用这个some别名来加载，用其他别名将会导致超时 `
require(['someOther'],function(some){....})非第一次加载some/v1.0，加载超时。应该采用与第一次加载时一致的别名some来加载.
require(['someDir/v1.0'],function(some){....})非第一次加载some/v1.0,加载超时。**应该采用与第一次加载时一致的别名some来加载.**

参考：http://requirejs.org/docs/errors.html#timeout。E文解释如下：
There was a script error in one of the listed modules. If there is no script error in the browser's error console, and if you are using Firebug, try loading the page in another browser like Chrome or Safari. Sometimes script errors do not show up in Firebug.
The path configuration for a module is incorrect. Check the "Net" or "Network" tab in the browser's developer tools to see if there was a 404 for an URL that would map to the module name. Make sure the script file is in the right place. In some cases you may need to use the paths configuration to fix the URL resolution for the script.
The paths config was used to set two module IDs to the same file, and that file only has one anonymous module in it. If module IDs "something" and "lib/something" are both configured to point to the same "scripts/libs/something.js" file, and something.js only has one anonymous module in it, this kind of timeout error can occur. The fix is to make sure all module ID references use the same ID (either choose "something" or "lib/something" for all references), or use map config.