title: restructuredtext与sphinx-doc1 

#  reStructuredText与sphinx-doc环境安装 
大家会发现，如果一个项目主要是用Python写的，其文档都很类似，比如：Python在线的HTML官方手册。这些项目的文档都来源于一个很不错的项目：Sphinx。这个Sphinx特指Sphinx doc这个项目（另一个也叫Sphinx的search的项目，虽然都叫一个名字）。
官网：http://sphinx-doc.org/
github:https://github.com/sphinx-doc/sphinx
以下出现Sphinx的地方，都特指Sphinx doc这个项目
##  使用场景 
很多开源Python库项目的API手册都是用这个写的，可以看Sphinx官网给的链接：http://sphinx-doc.org/examples.html
如果是用Python写的商业软件，也可以用这个来写技术文档，纯文本管理研发文档，保证功能代码、测试代码、相关文档同时更新
直接拿来写在线书。比如，这个《软件构建实践系列》就是：https://github.com/akun/pm
直接用来做slide等演示幻灯片，从一定程度上替代PowerPoint。比如，http://example.zhengkun.info/slide.html
##  功能 
这里就列举个人关心的几个特性：
文本是rst格式语法
生成HTML、PDF、Slide（需要插件）等格式的文档
支持文档、代码等引用
支持自定义样式
支持插件扩展
直接看官网手册了解更多：http://sphinx-doc.org/contents.html
##  安装 
http://www.sphinx-doc.org/en/stable/tutorial.html
```

pip install -U sphinx

```
创建一个目录。在这个目录下执行
sphinx-quickstart
然后运行make html即可。

##  中文搜索支持 
http://www.chenyudong.com/archives/sphinx-doc-support-chinese-search.html
现在sphinx-doc的master或者v1.4版本后就支持中文（简体+繁体）搜索了。
sphinx-doc的中文搜索是依靠jieba这个开源的类库来实现的，这个类库就支持简体和繁体的切分，所以就很容易实现了。
第一，你的系统需要安装jieba类库， ` pip install jieba `
第二，接下来修改sphinx的conf.py文件，为项目设置为中文的搜索配置。
```

# Language to be used for generating the HTML full-text search index.
# Sphinx supports the following languages:
#   'da', 'de', 'en', 'es', 'fi', 'fr', 'hu', 'it', 'ja'
#   'nl', 'no', 'pt', 'ro', 'ru', 'sv', 'tr', 'zh'
html_search_language = 'zh'

```
第三，可选配置
```

# A dictionary with options for the search language support, empty by default.
# 'ja' uses this config value.
# 'zh' user can custom change `jieba` dictionary path.
# html_search_options = {'dict': '/usr/lib/jieba.txt'}   # 根据需要设置jieba的词典路径

```
第四，接下来重新编译生成文档。` make html `

###  sphinx的基本搜索原理 
在编译的时候：
先对文本进行切词，把中文切分，
然后制作一个大的map对象，把关键字做为key，url做为value，保存到js文件。
搜索的时候寻找对应的关键字key，拿到url作为列表展示。
##  生成HTML 
很简单，默认支持就很好。
make html
python -m SimpleHTTPServer 9527
直接浏览器访问9527端口，就可以看到类似Python官方文档的效果。
##  生成PDF（不支持Python3） 
安装依赖库
```

pip install rst2pdf

```
编辑conf.py，增加或修改如下配置：
```

extensions = ['rst2pdf.pdfbuilder']

# -- Options for PDF output --------------------------------------------------
# Grouping the document tree into PDF files. List of tuples
# (source start file, target name, title, author, options).
#
# If there is more than one author, separate them with \\.
# For example: r'Guido van Rossum\\Fred L. Drake, Jr., editor'
#
# The options element is a dictionary that lets you override
# this config per-document.
# For example,
# ('index', u'MyProject', u'My Project', u'Author Name',
#  dict(pdf_compressed = True))
# would mean that specific document would be compressed
# regardless of the global pdf_compressed setting.
pdf_documents = [
    ('index', u'WebSOC', u'WebSOC Documentation', u'WebSOC Team Knownsec'),
]
# A comma-separated list of custom stylesheets. Example:
#pdf_stylesheets = ['sphinx','kerning','a4']
pdf_stylesheets = ['chinese','a4']
# A list of folders to search for stylesheets. Example:
pdf_style_path = ['.', '_styles']
# Create a compressed PDF
# Use True/False or 1/0
# Example: compressed=True
#pdf_compressed = False
# A colon-separated list of folders to search for fonts. Example:
# pdf_font_path = ['/usr/share/fonts', '/usr/share/texmf-dist/fonts/']
# Language to be used for hyphenation support
#pdf_language = "en_US"
# Mode for literal blocks wider than the frame. Can be
# overflow, shrink or truncate
#pdf_fit_mode = "shrink"
# Section level that forces a break page.
# For example: 1 means top-level sections start in a new page
# 0 means disabled
#pdf_break_level = 0
# When a section starts in a new page, force it to be 'even', 'odd',
# or just use 'any'
#pdf_breakside = 'any'
# Insert footnotes where they are defined instead of
# at the end.
#pdf_inline_footnotes = True
# verbosity level. 0 1 or 2
#pdf_verbosity = 0
# If false, no index is generated.
#pdf_use_index = True
# If false, no modindex is generated.
#pdf_use_modindex = True
# If false, no coverpage is generated.
#pdf_use_coverpage = True
# Name of the cover page template to use
#pdf_cover_template = 'sphinxcover.tmpl'
# Documents to append as an appendix to all manuals.
#pdf_appendices = []
# Enable experimental feature to split table cells. Use it
# if you get "DelayedTable too big" errors
#pdf_splittables = False
# Set the default DPI for images
#pdf_default_dpi = 72
# Enable rst2pdf extension modules (default is only vectorpdf)
# you need vectorpdf if you want to use sphinx's graphviz support
#pdf_extensions = ['vectorpdf']
# Page template name for "regular" pages
#pdf_page_template = 'cutePage'
# Show Table Of Contents at the beginning?
#pdf_use_toc = True
# How many levels deep should the table of contents be?
pdf_toc_depth = 9999
# Add section number to section references
pdf_use_numbered_links = False
# Background images fitting mode
pdf_fit_background_mode = 'scale'

```
编辑Makefile，增加如下代码：
Linux下的Makefie：
```

pdf:
	$(SPHINXBUILD) -b pdf $(ALLSPHINXOPTS) _build/pdf
	@echo
	@echo "Build finished. The PDF files are in _build/pdf."

```
Windows下的批处理：
```

if "%1" == "pdf" (
	%SPHINXBUILD% -b pdf %ALLSPHINXOPTS% %BUILDDIR%/pdf
	echo.
echo.Build finished. The PDF files are in %BUILDDIR%/pdf
	goto end
)

```
执行生成PDF
` make pdf `
python -m SimpleHTTPServer 9527
有关PDF的更多配置，可以阅读这个文档：http://ralsina.me/static/manual.pdf
##  生成Slide 
Slide就是我们常说的演示文档，如：Windows下的PowerPoint（PPT）；Mac下Keynote等等。这里用Sphinx生成在线的HTML5形式的Slide，操作也相对简单，也是需要依赖库和简单修改下配置。
安装依赖库
` pip install hieroglyph `
编辑conf.py，修改如下配置：
extensions = ['hieroglyph']
编辑Makefile，增加如下代码：
Linux下的Makefie：
```

slides:
	$(SPHINXBUILD) -b slides $(ALLSPHINXOPTS) $(BUILDDIR)/slides
	@echo
	@echo "Build finished. The slides are in $(BUILDDIR)/slides."

```
执行生成Slides
```

make slides
python -m SimpleHTTPServer 9527

```
有关Slide的更多信息，可以直接查看这个项目：https://github.com/nyergler/hieroglyph
##  sphinx_rtd_theme主题 
https://github.com/snide/sphinx_rtd_theme
$ pip install sphinx_rtd_theme
import sphinx_rtd_theme
编辑` conf.py ` file:
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]

参考:http://pm.readthedocs.org/doc/sphinx.html
