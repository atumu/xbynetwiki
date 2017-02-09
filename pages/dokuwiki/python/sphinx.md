title: sphinx 

#  自动生成api文档 

##  使用sphinx autodoc功能 
pip install Sphinx

###  快速生成默认配置方式 
进入你代码所在的目录,输入以下命令
sphinx-apidoc -F -o doc . 
cd doc
conf.py配置修改,取消注释，并把sys.path.insert(0, os.path.abspath('your_project_path'))，这里我改为.. 因为就在doc/config.py的上级目录
import os
import sys
sys.path.insert(0, os.path.abspath('..')
最后生成html
make html
用浏览器打开doc/_build/html/index.html即可。

###  使用交互式配置方式 
其实你大可以通过编辑快速方式生成的config.py进行配置。而没有必要回答这么一大堆问题。不过对于第一次使用，还是可以看看的。
进入你代码所在的目录,输入以下命令
sphinx-quickstart
会出来一系列要选填的东西：
```

> Root path for the documentation [.]:	doc
> Separate source and build directories (y/N) [n]:	y
> Name prefix for templates and static dir [_]:	<ENTER>
> Project name:	项目名
> Author name(s):	作者名
> Project version:	版本
> Project release [0.0.1]:	版本
> Source file suffix [.rst]:	<ENTER>
> Name of your master document (without suffix) [index]:	<ENTER>
> autodoc: automatically insert docstrings from modules (y/N) [n]:	y
> doctest: automatically test code snippets in doctest blocks (y/N) [n]:	n
> intersphinx: link between Sphinx documentation of different projects (y/N) [n]:	y
> todo: write “todo” entries that can be shown or hidden on build (y/N) [n]:	n
> coverage: checks for documentation coverage (y/N) [n]:	n
> pngmath: include math, rendered as PNG images (y/N) [n]:	n
> jsmath: include math, rendered in the browser by JSMath (y/N) [n]:	n
> ifconfig: conditional inclusion of content based on config values (y/N) [n]:	y
> Create Makefile? (Y/n) [y]:	y
> Create Windows command file? (Y/n) [y]:	y

```
还有个语言设置，如果是中文注释：zh_CN

路径我希望是在当面目录的 doc 目录里,所以我加入了 doc
项目名称和作者你就别照着我的填了.
autodoc 要改成y
其他的看你的需要来改就可以了.
输错了没关系.到conf.py自己改就可以了.
从 docstring 生成 doc

到**代码目录,执行**以下命令
sphinx-apidoc  -o doc .              #sphinx-apidoc [options] -o outputdir packagedir [pathnames]  
进入 doc 执行
cd doc
doc/source/conf.py配置修改,取消注释，并把sys.path.insert(0, os.path.abspath('your project path'))
import os
import sys
sys.path.insert(0, os.path.abspath('your project path')

最后生成html
make html
用浏览器打开doc/_build/html/index.html即可。


----
##  附录 

###  设置流行的主题 
` 设置主题 `
推荐使用readthedoc使用的主题，美观又简洁大方。首先安装主题库：
pip install sphinx_rtd_theme
然后配置conf.py:
```

import sphinx_rtd_theme
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]


```

###  phinx-apidoc命令行选项： 
$ sphinx-apidoc [options] -o outputdir packagedir [pathnames]  
```

Options:  
  -h, --help            show this help message and exit  
  -o DESTDIR, --output-dir=DESTDIR  
                        Directory to place all output  
  -d MAXDEPTH, --maxdepth=MAXDEPTH  
                        Maximum depth of submodules to show in the TOC  
                        (default: 4)  
  -f, --force           Overwrite all files  
  -n, --dry-run         Run the script without creating files  
  -T, --no-toc          Don't create a table of contents file  
  -s SUFFIX, --suffix=SUFFIX  
                        file suffix (default: rst)  
  -F, --full            Generate a full project with sphinx-quickstart  
  -H HEADER, --doc-project=HEADER  
                        Project name (default: root module name)  
  -A AUTHOR, --doc-author=AUTHOR  
                        Project author(s), used when --full is given  
  -V VERSION, --doc-version=VERSION  
                        Project version, used when --full is given  
  -R RELEASE, --doc-release=RELEASE  
                        Project release, used when --full is given, defaults  
                        to --doc-version 

```
