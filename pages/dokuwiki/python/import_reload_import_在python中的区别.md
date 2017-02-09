title: import_reload_import_在python中的区别 

#  import,reload,__import__在python中的区别 
import作用：导入/引入一个python标准模块，其中包括.py文件、带有__init__.py文件的目录。
多次重复使用import语句时，不会重新加载被指定的模块，只是把对该模块的内存地址给引用到本地变量环境。

reload函数作用：对已经加载的模块进行重新加载，一般用于原模块有变化等特殊情况，reload前该模块必须已经import过。reload会重新加载已加载的模块，但原来已经使用的实例还是会使用旧的模块，而新生产的实例会使用新的模块。reload后还是用原来的内存地址；不能支持from。。import。。格式的模块进行重新加载。

&lt;nowiki&gt;__import__函数作用：同import语句同样的功能，但__import__是一个函数&lt;/nowiki&gt;，并且只接收字符串作为参数。其实import语句就是调用这个函数进行导入工作的
```

import sys <==>sys = __import__('sys')
e.g：
__import__(module_name[, globals[, locals[, fromlist]]]) #可选参数默认为globals(),locals(),[]
__import__('os')    
__import__('os',globals(),locals(),['path','pip'])  #等价于from os import path, pip

```
通常**在动态加载时**可以使用到这个函数，比如你希望加载某个文件夹下的所用模块，但是其下的模块名称又会经常变化时，就可以使用这个函数动态加载所有模块了，**最常见的场景就是插件功能的支持。**

例如：插件规范取名为*-plugin.py,然后我们就可以通过&lt;nowiki&gt;__import__&lt;/nowiki&gt;导入所有插件了。
```

import glob,os  
  
modules = []  
for module_file in glob.glob("*-plugin.py"):  
    try:  
       module_name,ext = os.path.splitext(os.path.basename(module_file))  
       module = __import__(module_name)  
       modules.append(module)  
    except ImportError:  
       pass #ignore broken modules  
    #say hello to all modules  
    for module in modules:  
       module.hello()  

```
获得特定函数 
```

def getfunctionbyname(module_name,function_name):  
    module = __import__(module_name)  
    return getattr(module,function_name)  

```

**实现延迟化的模块导入** 
```

def import_string(import_name, silent=False):
    """Imports an object based on a string.  This is useful if you want to
    use import paths as endpoints or something similar.  An import path can
    be specified either in dotted notation (``xml.sax.saxutils.escape``)
    or with a colon as object delimiter (``xml.sax.saxutils:escape``).

    If `silent` is True the return value will be `None` if the import fails.

    :param import_name: the dotted name for the object to import.
    :param silent: if set to `True` import errors are ignored and
                   `None` is returned instead.
    :return: imported object
    """
    # force the import name to automatically convert to strings
    # __import__ is not able to handle unicode strings in the fromlist
    # if the module is a package
    import_name = str(import_name).replace(':', '.')
    try:
        try:
            __import__(import_name)
        except ImportError:
            if '.' not in import_name:
                raise
        else:
            return sys.modules[import_name]

        module_name, obj_name = import_name.rsplit('.', 1)
        try:
            module = __import__(module_name, None, None, [obj_name])
        except ImportError:
            # support importing modules not yet set up by the parent module
            # (or package for that matter)
            module = import_string(module_name)

        try:
            return getattr(module, obj_name)
        except AttributeError as e:
            raise ImportError(e)

    except ImportError as e:
        if not silent:
            reraise(
                ImportStringError,
                ImportStringError(import_name, e),
                sys.exc_info()[2])

```

示例：资源注册类：
```

class SourceRegistry(object):
    """ Central Registry for all available datasource engines"""
    
    sources = {}
    
    @classmethod
    def register_sources(cls, datasource_config):
        for module_name, class_names in datasource_config.items():
            module_obj = __import__(module_name, fromlist=class_names)
            for class_name in class_names:
                source_class = getattr(module_obj, class_name)
                cls.sources[source_class.type] = source_class


```

**扩展：通过字符串动态重新加载模块实现**
既然可以通过字符串来动态导入模块，那么是否可以通过字符串动态重新加载模块吗？试试reload('os')直接报错，是不是没有其他方式呢?
虽然不能直接reload但是可以先unimport(注意：并没有这个关键字)一个模块，然后再__import__来重新加载模块。
现在看看unimport操作如何实现，在Python解释里可以通过**globals(),locals(),vars(),dir()**等函数查看到当前环境下加载的模块及其位置，但是这些都是只读的；不过除此之外还有一个地方是专门存放模块的，这就是**sys.modules**，通过sys.modules可以查看所有的已加载并且成功的模块。
```

import sys  
__import__('a')      #第一次导入会打印消息  
del sys.modules['a']   #unimport  
__import__('a')    #再次导入还是会打印消息，因为已经unimport一次了  
__import__('a')    #这次就不会打印消息了  

```




参考：http://blog.csdn.net/five3/article/details/7762870
http://david-je.iteye.com/blog/1756788
http://blog.csdn.net/pi9nc/article/details/26750487