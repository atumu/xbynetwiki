title: whooshapi 

#  WhooshAPI学习 
##  whoosh.searching.Hit 
whoosh.searching.Hit(results, docnum, pos=None, score=None)
Represents a single search result (“hit”) in a Results object.
Hit表现得像字典一样，可以访问**被存储的field**(没有stored的field是没有的)，如果需要一个实际的dict对象，使用**Hit.fields()**返回。
```

>>> r = searcher.search(query.Term("content", "render"))
>>> r[0]
< Hit {title = u"Rendering the scene"} >
>>> r[0].rank
0
>>> r[0].docnum == 4592
True
>>> r[0].score
2.52045682
>>> r[0]["title"]
"Rendering the scene"
>>> r[0].keys()
["title"]

```

  * fields()：Returns a **dictionary** of the **stored fields** of the document this object represents.
  * highlights(fieldname, text=None, top=3, minscore=1) Returns highlighted snippets from the given field:
  * matched_terms()：Returns the set of ("fieldname", "text") tuples representing terms from the query that matched in this document.
只有使用 **terms=True** in the search call to record matching terms. 否则抛出异常。
```

>>> q = myparser.parse("alfa OR bravo OR charlie")
>>> results = searcher.search(q, terms=True)
>>> for hit in results:
...   print(hit["title"])
...   print("Contains:", hit.matched_terms())
...   print("Doesn't contain:", q.all_terms() - hit.matched_terms())

```

  * more_like_this(fieldname, text=None, top=10, numterms=5, model=<class 'whoosh.classify.Bo1Model'>, normalize=True, filter=None)
Returns a new Results object containing documents similar to this hit, based on “key terms” in the given field:
```

r = searcher.search(myquery)
for hit in r:
    print(hit["title"])
    print("Top 3 similar documents:")
    for subhit in hit.more_like_this("content", top=3):
      print("  ", subhit["title"])

```
