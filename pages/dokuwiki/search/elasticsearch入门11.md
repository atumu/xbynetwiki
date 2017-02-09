title: elasticsearch入门11 

#  Elasticsearch入门之嵌套 
###  嵌套对象 
事实上**在Elasticsearch中，创建丶删除丶修改一个文档是是原子性的，**因此我们可以在一个文档中储存密切关联的实体。举例来说，我们可以在一个文档中储存一笔订单及其所有内容，或是储存一个Blog文章及其所有回应，藉由传递一个comments阵列：
PUT /my_index/blogpost/1
```

{
  "title": "Nest eggs",
  "body":  "Making your money work...",
  "tags":  [ "cash", "shares" ],
  "comments": [ <1>
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}

```
如果我们依靠动态映射，comments栏位会被自动建立为一个object栏位。
因为所有内容都在同一个文档中，使搜寻时并不需要连接(join)blog文章与回应，因此搜寻表现更加优异。
问题在於以上的文档可能会如下所示的匹配一个搜寻：
GET /_search
```

{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "Alice" }},
        { "match": { "age":  28      }} <1>
      ]
    }
  }
}

```
Alice是31岁，而不是28岁！
造成跨对象配对的原因如同我们在对象阵列中所讨论到，在于我们优美结构的JSON文档在索引中被扁平化为下方的 键-值 形式：
{
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ],
  "comments.name":    [ alice, john, smith, white ],
  "comments.comment": [ article, great, like, more, please, this ],
  "comments.age":     [ 28, 31 ],
  "comments.stars":   [ 4, 5 ],
  "comments.date":    [ 2014-09-01, 2014-10-22 ]
}
Alice与31 以及 John与2014-09-01 之间的关联已经无法挽回的消失了。 当object类型的栏位用于储存_单一_对象是非常有用的。 从搜寻的角度来看，对於排序一个对象阵列来说关联是不需要的东西。
**这是_嵌套对象_被设计来解决的问题。 藉由映射commments栏位为nested类型而不是object类型， 每个嵌套对象会被索引为一个_隐藏分割文档_，例如：**
{ <1>
  "comments.name":    [ john, smith ],
  "comments.comment": [ article, great ],
  "comments.age":     [ 28 ],
  "comments.stars":   [ 4 ],
  "comments.date":    [ 2014-09-01 ]
}
{ <2>
  "comments.name":    [ alice, white ],
  "comments.comment": [ like, more, please, this ],
  "comments.age":     [ 31 ],
  "comments.stars":   [ 5 ],
  "comments.date":    [ 2014-10-22 ]
}
{ <3>
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ]
}
第一个嵌套对象 第二个嵌套对象 根或是父文档
藉由分别索引每个嵌套对象，对象的栏位中保持了其关联。 我们的查询可以只在同一个嵌套对象都匹配时才回应。
不仅如此，因嵌套对象都被索引了，连接嵌套对象至根文档的查询速度非常快--几乎与查询单一文档一样快。
这些额外的嵌套对象被隐藏起来，我们无法直接访问他们。 为了要新增丶修改或移除一个嵌套对象，我们必须重新索引整个文档。 **要牢记搜寻要求的结果并不是只有嵌套对象，而是整个文档。**

##  嵌套对象映射 
设定一个` nested栏位 `很简单- -在你会设定为object类型的地方，改为nested类型：
PUT /my_index
```

{
  "mappings": {
    "blogpost": {
      "properties": {
        "comments": {
          "type": "nested", <1>
          "properties": {
            "name":    { "type": "string"  },
            "comment": { "type": "string"  },
            "age":     { "type": "short"   },
            "stars":   { "type": "short"   },
            "date":    { "type": "date"    }
          }
        }
      }
    }
  }
}

```
一个nested栏位接受与object类型相同的参数。
所需仅此而已。 任何comments对象会被索引为分离嵌套对象。 

##  查询嵌套对象 
**因嵌套对象(nested objects)会被索引为分离的隐藏文档，我们不能直接查询它们。而是使用 nested查询或 nested 过滤器来存取它们：**
GET /my_index/blogpost/_search
```

{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }}, <1>
        {
          "nested": {
            "path": "comments", <2>
            "query": {
              "bool": {
                "must": [ <3>
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age":  28     }}
                ]
        }}}}
      ]
}}}

```
**title条件运作在根文档上 nested条件深入嵌套的comments栏位。**它不会在存取根文档的栏位，或是其他嵌套文档的栏位。 comments.name以及comments.age运作在相同的嵌套文档。
一个nested栏位可以包含其他nested栏位。 相同的，一个nested查询可以包含其他nested查询。 嵌套阶层会如同你预期的运作。
当然，一个nested查询可以匹配多个嵌套文档。 每个文档的匹配会有各自的关联分数，但多个分数必须减少至单一分数才能应用至根文档。
在预设中，它会平均所有嵌套文档匹配的分数。**这可以藉由设定score_mode参数为avg, max, sum或甚至none(为了防止根文档永远获得1.0的匹配分数时)来控制。**
GET /my_index/blogpost/_search
```

{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }},
        {
          "nested": {
            "path":       "comments",
            "score_mode": "max", <1>
            "query": {
              "bool": {
                "must": [
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age":  28     }}
                ]
        }}}}
      ]
}}}

```
从最匹配的嵌套文档中给予根文档的_score值。
注意
nested过滤器类似於nested查询，除了无法使用score_mode参数。 只能使用在_filter context_—例如在filtered查询中--其作用类似其他的过滤器： 包含或不包含，但不评分。
nested过滤器的结果本身不会缓存，通常缓存规则会被应用於nested过滤器_之中_的过滤器。

##  以嵌套栏位排序 
我们可以依照嵌套栏位中的值来排序，甚至藉由分离嵌套文档中的值。为了使其结果更加有趣，我们加入另一个记录：
PUT /my_index/blogpost/2
```

{
  "title": "Investment secrets",
  "body":  "What they don't tell you ...",
  "tags":  [ "shares", "equities" ],
  "comments": [
    {
      "name":    "Mary Brown",
      "comment": "Lies, lies, lies",
      "age":     42,
      "stars":   1,
      "date":    "2014-10-18"
    },
    {
      "name":    "John Smith",
      "comment": "You're making it up!",
      "age":     28,
      "stars":   2,
      "date":    "2014-10-16"
    }
  ]
}

```
想像我们要取回在十月中有收到回应的blog文章，并依照所取回的各个blog文章中最少stars数量的顺序作排序。 这个搜寻请求如下：
GET /_search
```

{
  "query": {
    "nested": { <1>
      "path": "comments",
      "filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  },
  "sort": {
    "comments.stars": { <2>
      "order": "asc",   <2>
      "mode":  "min",   <2>
      "nested_filter": { <3>
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  }
}

```
nested查询限制了结果为十月份收到回应的blog文章。 结果在所有匹配的回应中依照comment.stars栏位的最小值(min)作递增(asc)的排序。 排序条件中的nested_filter与主查询query条件中的nested查询相同。 於下一个下方解释。
**为什么我们要在nested_filter重复写上查询条件？ 原因是排序在於执行查询后才发生。** 此查询匹配了在十月中有收到回应的blog文章，回传blog文章文档作为结果。 如果我们不加上nested_filter条件，我们最後会依照任何blog文章曾经收到过的回应作排序，而不是在十月份收到的。

##  嵌套-集合 
如同我们在查询时需要使用nested查询来存取嵌套对象，专门的nested集合使我们可以取得嵌套对象中栏位的集合：
GET /my_index/blogpost/` _search?search_type=count `
```

{
  "aggs": {
    "comments": { <1>
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_month": {
          "date_histogram": { <2>
            "field":    "comments.date",
            "interval": "month",
            "format":   "yyyy-MM"
          },
          "aggs": {
            "avg_stars": {
              "avg": { <3>
                "field": "comments.stars"
              }
            }
          }
        }
      }
    }
  }
}

```
nested集合深入嵌套对象的comments栏位 评论基於comments.date栏位被分至各个月份分段 每个月份分段单独计算星号的平均数
结果显示集合发生於嵌套文档层级：
...
```

"aggregations": {
  "comments": {
     "doc_count": 4, <1>
     "by_month": {
        "buckets": [
           {
              "key_as_string": "2014-09",
              "key": 1409529600000,
              "doc_count": 1, <1>
              "avg_stars": {
                 "value": 4
              }
           },
           {
              "key_as_string": "2014-10",
              "key": 1412121600000,
              "doc_count": 3, <1>
              "avg_stars": {
                 "value": 2.6666666666666665
              }
           }
        ]
     }
  }
}
...

```
此处总共有四个comments: 一个在九月以及三个在十月

##  反向嵌套-集合 
一个nested集合只能存取嵌套文档中的栏位，而无法看见根文档或其他嵌套文档中的栏位。 然而，我们可以_跳出_嵌套区块，藉由reverse_nested集合回到父阶层。
举例来说，我们可以发现使用评论者的年龄为其加上tags很有趣。 comment.age是在嵌套栏位中，但是tags位於根文档：
GET /my_index/blogpost/_search?search_type=count
```

{
  "aggs": {
    "comments": {
      "nested": { <1>
        "path": "comments"
      },
      "aggs": {
        "age_group": {
          "histogram": { <2>
            "field":    "comments.age",
            "interval": 10
          },
          "aggs": {
            "blogposts": {
              "reverse_nested": {}, <3>
              "aggs": {
                "tags": {
                  "terms": { <4>
                    "field": "tags"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}

```
nested集合深入comments对象 histogram集合以comments.age栏位聚集成每十年一个的分段 reverse_nested集合跳回到根文档 terms集合计算每个年龄分段的火红词语
简略的结果显示如下：

..
```

"aggregations": {
  "comments": {
     "doc_count": 4, <1>
     "age_group": {
        "buckets": [
           {
              "key": 20, <2>
              "doc_count": 2, <2>
              "blogposts": {
                 "doc_count": 2, <3>
                 "tags": {
                    "doc_count_error_upper_bound": 0,
                    "buckets": [ <4>
                       { "key": "shares",   "doc_count": 2 },
                       { "key": "cash",     "doc_count": 1 },
                       { "key": "equities", "doc_count": 1 }
                    ]
                 }
              }
           },

```
...
共有四个评论 有两个评论的发表者年龄介於20至30之间 两个blog文章与这些评论相关 这些blog文章的火红标签是shares丶cash丶equities
什麽时候要使用嵌套对象
嵌套对象对於当有一个主要实体(如blogpost)，加上有限数量的紧密相关实体(如comments)是非常有用的。 有办法能以评论内容找到blog文章很有用，且nested查询及过滤器提供短查询时间连接(fast query-time joins)。
嵌套模型的缺点如下：
如欲新增丶修改或删除一个嵌套文档，则必须重新索引整个文档。因此越多嵌套文档造成越多的成本。
搜寻请求回传整个文档，而非只有匹配的嵌套文档。 虽然有个进行中的计画要支持只回传根文档及最匹配的嵌套文档，但目前并未支持。
有时你需要完整分离主要文档及其关连实体。 父-子关系提供这一个功能。