title: whoosh5 

#  Whoosh高亮结果与Hit的一些坑 
Schema对于DATETIME field,add_document时一定要是datetime对象。
对于不是stored=True的field不会出现在results的hit中，也就无法读取。
在高亮时，如果想要某些没有关键字，但是需要显示的field值。此时需要设置WholeFragmenter与minscore=0,如下：

```

 hf = HtmlFormatter(tagname="code", classname="match", termclass="term")
fragmenter=WholeFragmenter(charlimit=None)
page = searcher.search_page(q, curPage, pagelen)#,terms=True
page.results.fragmenter=fragmenter
page.results.formatter = hf
resPage=dict(pagenum=page.pagenum,pagecount=page.pagecount,total=page.total,posts=[])
for hint in page:
	tmp=dict()
	tmp['title']=hint.highlights("title",minscore=0)
	tmp['author']=hint["author"]
	tmp['location']=hint["location"]
	tmp['summary']=hint.highlights("summary",minscore=0)#hint["content"].replace(key,"<code >%s< /code>" % key)

	resPage['posts'].append(tmp)

```