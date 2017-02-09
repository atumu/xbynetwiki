title: python-markdown 

#  Python-Markdown 
官网：https://pythonhosted.org/Markdown/index.html
当前版本:2.6.7

pip install markdown


markdown语法：http://daringfireball.net/projects/markdown/
convert Markdown syntax into HTML.

```

import markdown
html = markdown.markdown(your_text_string，output_format='html5')

```
(markdown.markdown and markdown.markdownFromFile)


##  命令行形式： 
```

$ python -m markdown input_file.txt
$ echo "Some **Markdown** text." | python -m markdown > output.html
$ python -m markdown --help

```

##  扩展： 
http://pythonhosted.org/Markdown/extensions/
Extension	“Name”
```

Extra	markdown.extensions.extra
    Abbreviations	markdown.extensions.abbr
    Attribute Lists	markdown.extensions.attr_list
    Definition Lists	markdown.extensions.def_list
    Fenced Code Blocks	markdown.extensions.fenced_code
    Footnotes	markdown.extensions.footnotes
    Tables	markdown.extensions.tables
    Smart Strong	markdown.extensions.smart_strong
Admonition	markdown.extensions.admonition
CodeHilite	markdown.extensions.codehilite
HeaderId	markdown.extensions.headerid
Meta-Data	markdown.extensions.meta
New Line to Break	markdown.extensions.nl2br
Sane Lists	markdown.extensions.sane_lists
SmartyPants	markdown.extensions.smarty
Table of Contents	markdown.extensions.toc
WikiLinks	markdown.extensions.wikilinks

```
```

markdown.markdown(some_text, extensions=[MyExtension(), 'path.to.my.ext', 'markdown.extensions.footnotes'])

```
```

$ python -m markdown -x markdown.extensions.footnotes -x markdown.extensions.tables input.txt > output.html

```


##  自定义扩展 
```

#encoding=utf-8  
##预处理器  
from markdown.preprocessors import Preprocessor  
class CodePreprocessor(Preprocessor):  
    def run(self, lines):  
        new_lines = []  
        flag_in = False  
        block = []  
        for line in lines:  
            if line[:3]=='!!!':                  
                flag_in = True  
                block.append('<pre class="brush: %s;">' % line[3:].strip())  
            elif flag_in:  
                if line.strip() and line[0]=='!':  
                    block.append(line[1:])  
                else:  
                    flag_in = False  
                    block.append('</pre>')  
                    block.append(line)  
                    new_lines.extend(block)  
                    block = []  
            else:  
                new_lines.append(line)  
        if not new_lines and block:  
            new_lines = block  
        return new_lines  
  
##后置处理器  
from markdown.postprocessors import Postprocessor  
class CodePostprocessor(Postprocessor):  
    def run(self, text):  
        t_list = []  
        for line in text.split('\n'):  
            if line[:5]=='<p>!<':  
                line = line.lstrip('<p>').replace('</p>', '')[1:]  
            t_list.append(line)   
        return '\n'.join(t_list)      
      
##扩展主体类          
from markdown.extensions import Extension  
from markdown.util import etree  
class CodeExtension(Extension):  
    def __init__(self, configs={}):  
        self.config = configs  
  
    def extendMarkdown(self, md, md_globals):  
        ##注册扩展，用于markdown.reset时扩展同时reset  
        md.registerExtension(self)     
                  
        ##设置Preprocessor  
        codepreprocessor = CodePreprocessor()  
        #print md.preprocessors.keys()  
        md.preprocessors.add('codepreprocessor', codepreprocessor, '<normalize_whitespace')  
          
        ##设置Postprocessor  
        codepostprocessor = CodePostprocessor()  
        #print md.postprocessors.keys()  
        md.postprocessors.add('codepostprocessor', codepostprocessor, '>unescape')  
          
        ##print md_globals   ##markdown全局变量  

```
```

#encoding=utf-8  
import markdown  
import markdowncode  
  
text = `  `' 
!!!python 
! 
!def foo(): 
 
###title 
'''  
  
configs = {}  
  
myext = markdowncode.CodeExtension(configs=configs)  
md = markdown.markdown(text, extensions=[myext])  
print md  

```
主要扩展了2个功能:
```

一个是把形如：
!!!python  
!  
!def foo():  
!  return 'foo'  
转换成：
<pre class="brush: python;">  
  
def foo():  
  return 'foo'  
</pre>

```

```

一个是把形如：
!<div>  
#h1  
!</div>  
转换成：
<div>  
<h1>h1</h1>  
</div>

```  
参考：https://pythonhosted.org/Markdown/index.html
http://blog.csdn.net/five3/article/details/18010021