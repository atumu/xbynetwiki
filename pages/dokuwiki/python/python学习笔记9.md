title: python学习笔记9 

#  Python学习笔记之模块进阶 
##  import如何工作 
  * 模块只会在**第一次导入**时才会运行其中的代码。
Python把已经载入的模块存储到一个名为sys.modules的内存表中。只有当其中不存在时才会执行导入操作，否则直接提取。
  * **模块搜索路径**：程序主目录-》PYTHONPATH目录-》标准链接库目录-任何` sys.path `路径的内容
  * 包导入与命名空间 __init__.py包文件。
  * import dir1.dir2.mod as module1
  * if __name__=='__main__'
  * 修改模块搜索路径:
```

import sys
print(sys.path)
sys.path.append('c:\\src')

```
##  模块内省(introspection) 
```

import m
m.__dic__['name']
sys.modules['m'].name
getattr(m,'name')
m.__name__

>>> import re
>>> re.__name__
're'
>>> re.__file__
'D:\\Program Files\\python3\\lib\\re.py'

>>> getattr(re,'__name__')
're'

```




