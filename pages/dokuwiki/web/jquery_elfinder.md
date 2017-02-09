title: jquery_elfinder 

#  jQuery elFinder基于web文件管理器 
elFinder 是一个基于 Web 的文件管理器，灵感来自 Mac OS X 的 Finder 程序。
官网：http://elfinder.org/
github:https://github.com/Studio-42/elFinder/wiki
![](/data/dokuwiki/web/pasted/20151106-220048.png)
依赖：jQueryUI,jQuery
```

<link rel="stylesheet" type="text/css" media="screen" href="http://ajax.googleapis.com/ajax/libs/jqueryui/1.8.14/themes/smoothness/jquery-ui.css" />
<script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.min.js" ></script> 
<script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jqueryui/1.8.14/jquery-ui.min.js"></script>

<link rel="stylesheet" type="text/css" media="screen" href="css/elfinder.min.css">
<script type="text/javascript" src="js/elfinder.min.js"></script>
<script type="text/javascript" src="js/i18n/elfinder.zh_CN.js"></script>

```

##  使用 
```

<script type="text/javascript" charset="utf-8">
    $().ready(function() {
        var elf = $('#elfinder').elfinder({
            // lang: 'ru',             // language (OPTIONAL)
            url : 'php/connector.php'  // connector URL (REQUIRED)
        }).elfinder('instance');            
    });
</script>

<!-- Element where elFinder will be created (REQUIRED) -->
<div id="elfinder"></div>

```
##  配置选项 
请参考：https://github.com/Studio-42/elFinder/wiki/Client-configuration-options