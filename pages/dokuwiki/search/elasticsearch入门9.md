title: elasticsearch入门9 

#  Elasticsearch入门之索引管理 
##  创建索引 
**迄今为止，我们简单的通过添加一个文档的方式创建了一个索引。这个索引使用默认设置，新的属性通过动态映射添加到分类中。**
**现在我们需要对这个过程有更多的控制：我们需要确保索引被创建在适当数量的分片上，在索引数据_之前_设置好分析器和类型映射。**
为了达到目标，我们需要**手动创建索引，在请求中加入所有设置和类型映射**，如下所示：
**PUT** /my_index
```

{
    "settings": { ... any settings ... },
    "mappings": {
        "type_one": { ... any mappings ... },
        "type_two": { ... any mappings ... },
        ...
    }

```
事实上，你可以通过在 config/` elasticsearch.yml ` 中添加下面的配置来**防止自动创建索引。**
action.auto_create_index: false
今后，我们将介绍怎样用【索引模板】来自动预先配置索引。这在索引日志数据时尤其有效： 你将日志数据索引在一个以日期结尾的索引上，第二天，一个新的配置好的索引会自动创建好。
##  删除索引 
使用以下的请求来删除索引：
DELETE /my_index
你也可以用下面的方式删除多个索引
DELETE /index_one,index_two
DELETE /index_*
你甚至可以删除所有索引
DELETE /_all

##  索引设置 
你可以通过很多种方式来自定义索引行为，你可以阅读Index Modules reference documentation，但是：
**提示: Elasticsearch 提供了优化好的默认配置。除非你明白这些配置的行为和为什么要这么做，请不要修改这些配置。**
下面是两个最重要的设置：
number_of_shards
定义一个索引的**主分片个数**，默认值是 `5`。这个配置在索引创建后不能修改。
number_of_replicas
每个主分片的**复制分片个数**，默认是 `1`。这个配置可以随时在活跃的索引上修改。
例如，我们可以创建只有一个主分片，没有复制分片的小索引。
```

PUT /my_temp_index
{
    "settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    }
}

```
然后，我们可以用 update-index-settings API **动态修改复制分片个数：**
` PUT ` /my_temp_index/` _settings `
```

{
    "number_of_replicas": 1
}

```


##  配置分析器 
第三个重要的索引设置是 analysis 部分，用来配置已存在的分析器或创建自定义分析器来定制化你的索引。
standard 分析器是用于全文字段的默认分析器，对于大部分西方语系来说是一个不错的选择。它考虑了以下几点：
  * standard 分词器，在词层级上分割输入的文本。
  * standard 表征过滤器，被设计用来整理分词器触发的所有表征（但是目前什么都没做）。
  * lowercase 表征过滤器，将所有表征转换为小写。
  * stop 表征过滤器，删除所有可能会造成搜索歧义的停用词，如 a，the，and，is。
默认情况下，停用词过滤器是被禁用的。如需启用它，你可以通过创建一个基于 standard 分析器的自定义分析器，并且设置 stopwords 参数。可以提供一个停用词列表，或者使用一个特定语言的预定停用词列表。
在下面的例子中，我们创建了一个新的分析器，叫做 es_std，并使用预定义的西班牙语停用词：
PUT /spanish_docs
```

{
    "settings": {
        "analysis": {
            "analyzer": {
                "es_std": {
                    "type":      "standard",
                    "stopwords": "_spanish_"
                }
            }
        }
    }
}

```
**es_std 分析器不是全局的，它仅仅存在于我们定义的 spanish_docs 索引中。为了用 analyze API 来测试它，我们需要使用特定的索引名。**

GET /spanish_docs/_analyze?` analyzer=es_std `
El veloz zorro marrón
下面简化的结果中显示停用词 El 被正确的删除了：
```

{
  "tokens" : [
    { "token" :    "veloz",   "position" : 2 },
    { "token" :    "zorro",   "position" : 3 },
    { "token" :    "marrón",  "position" : 4 }
  ]
}

```


自定义分析器

虽然 Elasticsearch 内置了一系列的分析器，但是真正的强大之处在于定制你自己的分析器。你可以通过在配置文件中组合字符过滤器，分词器和表征过滤器，来满足特定数据的需求。

在 【分析器介绍】 中，我们提到 分析器 是三个顺序执行的组件的结合（字符过滤器，分词器，表征过滤器）。

字符过滤器

字符过滤器是让字符串在被分词前变得更加“整洁”。例如，如果我们的文本是 HTML 格式，它可能会包含一些我们不想被索引的 HTML 标签，诸如 <p> 或 <div>。

我们可以使用 html_strip 字符过滤器 来删除所有的 HTML 标签，并且将 HTML 实体转换成对应的 Unicode 字符，比如将 &Aacute; 转成 Á。

一个分析器可能包含零到多个字符过滤器。
分词器

一个分析器 必须 包含一个分词器。分词器将字符串分割成单独的词（terms）或表征（tokens）。standard 分析器使用 standard 分词器将字符串分割成单独的字词，删除大部分标点符号，但是现存的其他分词器会有不同的行为特征。

例如，keyword 分词器输出和它接收到的相同的字符串，不做任何分词处理。[whitespace 分词器]只通过空格来分割文本。[pattern 分词器]可以通过正则表达式来分割文本。
表征过滤器

分词结果的 表征流 会根据各自的情况，传递给特定的表征过滤器。

表征过滤器可能修改，添加或删除表征。我们已经提过 lowercase 和 stop 表征过滤器，但是 Elasticsearch 中有更多的选择。stemmer 表征过滤器将单词转化为他们的根形态（root form）。ascii_folding 表征过滤器会删除变音符号，比如从 très 转为 tres。 ngram 和 edge_ngram 可以让表征更适合特殊匹配情况或自动完成。
在【深入搜索】中，我们将举例介绍如何使用这些分词器和过滤器。但是首先，我们需要阐述一下如何创建一个自定义分析器

##  创建自定义分析器 
与索引设置一样，我们预先配置好 es_std 分析器，我们可以再 analysis 字段下配置字符过滤器，分词器和表征过滤器：
PUT /my_index
```

{
    "settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
}

```
作为例子，我们来配置一个这样的分析器：
用 html_strip 字符过滤器去除所有的 HTML 标签
将 & 替换成 and，使用一个**自定义的 mapping 字符过滤器**
```

"char_filter": {
    "&_to_and": {
        "type":       "mapping",
        "mappings": [ "&=> and "]
    }
}

```
使用 standard 分词器分割单词
使用 lowercase 表征过滤器将词转为小写
用 stop 表征过滤器去除一些自定义停用词。
```

"filter": {
    "my_stopwords": {
        "type":        "stop",
        "stopwords": [ "the", "a" ]
    }
}

```
根据以上描述来将预定义好的分词器和过滤器组合成我们的分析器：
```

"analyzer": {
    "my_analyzer": {
        "type":           "custom",
        "char_filter":  [ "html_strip", "&_to_and" ],
        "tokenizer":      "standard",
        "filter":       [ "lowercase", "my_stopwords" ]
    }
}

```
用下面的方式可以将以上请求合并成一条：
PUT /my_index
```

{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}

```
创建索引后，用 analyze API 来测试新的分析器：
GET /my_index/_analyze?` analyzer=my_analyzer `
The quick & brown fox
下面的结果证明我们的分析器能正常工作了：
```

{
  "tokens" : [
      { "token" :   "quick",    "position" : 2 },
      { "token" :   "and",      "position" : 3 },
      { "token" :   "brown",    "position" : 4 },
      { "token" :   "fox",      "position" : 5 }
    ]
}

```
**除非我们告诉 Elasticsearch 在哪里使用，否则分析器不会起作用。**我们可以通过下面的映射将它应用在一个 string 类型的字段上：
PUT /my_index/_mapping/my_type
```

{
    "properties": {
        "title": {
            "type":      "string",
            "analyzer":  "my_analyzer"
        }
    }
}

```

##  类型和映射 
**类型 在 Elasticsearch 中表示一组相似的文档。**
_类型_ 由一个 名称（比如 user 或 blogpost）和一个类似数据库表结构的映射组成，描述了文档中可能包含的每个字段的 属性，数据类型（比如 string, integer 或 date），和是否这些字段需要被 Lucene 索引或储存。
在【文档】一章中，我们说过类型类似关系型数据库中的表格，一开始你可以这样做类比，但是现在值得更深入阐释一下什么是类型，且在 Lucene 中是怎么实现的。
**Lucene 如何处理文档**
Lucene 中，一个文档由一组简单的键值对组成，一个字段至少需要有一个值，但是任何字段都可以有多个值。类似的，一个单独的字符串可能在分析过程中被转换成多个值。Lucene 不关心这些值是字符串，数字或日期，所有的值都被当成 不透明字节
当我们在 Lucene 中索引一个文档时，每个字段的值都被加到相关字段的倒排索引中。你也可以选择将原始数据 储存 起来以备今后取回。
类型是怎么实现的
**Elasticsearch 类型是在这个简单基础上实现的。一个索引可能包含多个类型，每个类型有各自的映射和文档，保存在同一个索引中。**
因为 **Lucene 没有文档类型的概念**，每个文档的类型名被储存在一个叫 _type 的元数据字段上。**当我们搜索一种特殊类型的文档时，Elasticsearch 简单的通过 _type 字段来过滤出这些文档。**
**Lucene 同样没有映射的概念。映射是 Elasticsearch 将复杂 JSON 文档_映射_成 Lucene 需要的扁平化数据的方式。**
例如，user 类型中 name 字段的映射声明这个字段是一个 string 类型，在被加入倒排索引之前，它的数据需要通过 whitespace 分析器来分析。
```

"name": {
    "type":     "string",
    "analyzer": "whitespace"
}

```
##  预防类型陷阱 
事实上不同类型的文档可以被加到同一个索引里带来了一些预想不到的困难。
想象一下我们的索引中有两种类型：blog_en 表示英语版的博客，blog_es 表示西班牙语版的博客。两种类型都有 title 字段，但是其中一种类型使用 english 分析器，另一种使用 spanish 分析器。
使用下面的查询就会遇到问题：
GET /_search
```

{
    "query": {
        "match": {
            "title": "The quick brown fox"
        }
    }
}

```
我们在两种类型中搜索 title 字段，首先需要分析查询语句，但是应该使用哪种分析器呢，spanish 还是 english？Elasticsearch 会采用第一个被找到的 title 字段使用的分析器，这对于这个字段的文档来说是正确的，但对另一个来说却是错误的。
我们可以通过给字段取不同的名字来避免这种错误 —— 比如，用 title_en 和 title_es。或者在查询中明确包含各自的类型名。
GET /_search
```

{
    "query": {
        "multi_match": { <1>
            "query":    "The quick brown fox",
            "fields": [ "blog_en.title", "blog_es.title" ]
        }
    }
}

```
multi_match 查询在多个字段上执行 match 查询并一起返回结果。
新的查询中 english 分析器用于 blog_en.title 字段，spanish 分析器用于 blog_es.title 字段，然后通过综合得分组合两种字段的结果。
这种办法对具有相同数据类型的字段有帮助，但是想象一下如果你将下面两个文档加入同一个索引，会发生什么：
类型: user
 { "login": "john_smith" }
类型: event
 { "login": "2014-06-01" }
**Lucene 不在乎一个字段是字符串而另一个字段是日期，它会一视同仁的索引这两个字段。**
然而，假如我们试图 排序 event.login 字段，Elasticsearch 需要将 login 字段的值加载到内存中。像我们在 【字段数据介绍】中提到的，它将 任意文档 的值加入索引而不管它们的类型。
它会尝试加载这些值为字符串或日期，取决于它遇到的第一个 login 字段。这可能会导致预想不到的结果或者以失败告终。
**提示：为了保证你不会遇到这些冲突，建议在同一个索引的每一个类型中，确保用_同样的方式_映射_同名_的字段**

##  根对象 
**映射的最高一层被称为 根对象**，它可能包含下面几项：
  * 一个 properties 节点，列出了文档中可能包含的每个字段的映射
  * 多个元数据字段，每一个都以下划线开头，例如 _type, _id 和 _source
  * 设置项，控制如何动态处理新的字段，例如 analyzer, dynamic_date_formats 和 dynamic_templates。
  * 其他设置，可以同时应用在根对象和其他 object 类型的字段上，例如 enabled, dynamic 和 include_in_all
  * 属性
我们已经在【核心字段】和【复合核心字段】章节中介绍过文档字段和属性的三个最重要的设置：
  * type： 字段的数据类型，例如 string 和 date
  * index： 字段是否应当被当成全文来搜索（analyzed），或被当成一个准确的值（not_analyzed），还是完全不可被搜索（no）
  * analyzer： 确定在索引和或搜索时全文字段使用的 分析器。
我们将在下面的章节中介绍其他字段，例如 ip, geo_point 和 geo_shape

##  文档 ID 
**文档唯一标识由四个元数据字段组成：**
  * _id：文档的字符串 ID
  * _type：文档的类型名
  * _index：文档所在的索引
  * _uid：_type 和 _id 连接成的 type#id
默认情况下，_uid 是被保存（可取回）和索引（可搜索）的。_type 字段被索引但是没有保存，_id 和 _index 字段则既没有索引也没有储存，它们并不是真实存在的。
尽管如此，你仍然可以像真实字段一样查询 _id 字段。Elasticsearch 使用 _uid 字段来追溯 _id。虽然你可以修改这些字段的 index 和 store 设置，但是基本上不需要这么做。
_id 字段有一个你可能用得到的设置：**` path 设置告诉 Elasticsearch 它需要从文档本身的哪个字段中生成 _id `**
```

PUT /my_index
{
    "mappings": {
        "my_type": {
            "_id": {
                "path": "doc_id" <1>
            },
            "properties": {
                "doc_id": {
                    "type":   "string",
                    "index":  "not_analyzed"
                }
            }
        }
    }
}

```
从 doc_id 字段生成 _id
然后，当你索引一个文档时：
```

POST /my_index/my_type
{
    "doc_id": "123"
}

```
**警告：虽然这样很方便，但是注意它对 bulk 请求（见【bulk 格式】）有个轻微的` 性能影响 `。处理请求的节点将不能仅靠解析元数据行来决定将请求分配给哪一个分片，而需要解析整个文档主体。**

##  动态映射 
当 Elasticsearch 遭遇一个位置的字段时，它通过【动态映射】来确定字段的数据类型且自动将该字段加到类型映射中。
幸运的是，你可以通过 dynamic 设置来控制这些行为，它接受下面几个选项：
  * true：自动添加字段（默认）
  * false：忽略字段
  * strict：当遇到未知字段时抛出异常
dynamic 设置可以用在根对象或任何 object 对象上。你可以将 dynamic 默认设置为 strict，而在特定内部对象上启用它：
PUT /my_index
```

{
    "mappings": {
        "my_type": {
            "dynamic":      "strict", <1>
            "properties": {
                "title":  { "type": "string"},
                "stash":  {
                    "type":     "object",
                    "dynamic":  true <2>
                }
            }
        }
    }
}

```
当遇到未知字段时，my_type 对象将会抛出异常

##  自定义动态索引 
如果你想在运行时的增加新的字段，你可能会开启动态索引。虽然有时动态映射的 规则 显得不那么智能，幸运的是我们可以通过设置来自定义这些规则。
**日期检测**
当 Elasticsearch 遇到一个新的字符串字段时，它会检测这个字段是否包含一个可识别的日期，**比如 2014-01-01。如果它看起来像一个日期，这个字段会被作为 date 类型添加，否则，它会被作为 string 类型添加。**
有些时候这个规则可能导致一些问题。想象你有一个文档长这样：
{ "note": "2014-01-01" }
假设这是**第一次见到 note 字段**，它会被添加为 date 字段，但是如果下一个文档像这样：
{ "note": "Logged out" }
**这显然不是一个日期，但为时已晚。**这个字段已经被添加为日期类型，这个 不合法的日期 将引发异常。
**日期检测可以通过在根对象上设置 date_detection 为 false 来关闭：**
```

PUT /my_index
{
    "mappings": {
        "my_type": {
            "date_detection": false
        }
    }
}

```
使用这个映射，字符串将始终是 string 类型。**假如你需要一个 date 字段，你得手动添加它。**
提示：
**Elasticsearch 判断字符串为日期的规则可以通过 ` dynamic_date_formats ` 配置 来修改。**

**动态模板**
使用 ` dynamic_templates `，你可以完全控制新字段的映射，你设置可以通过字段名或数据类型应用一个完全不同的映射。
每个模板都有一个名字用于描述这个模板的用途，一个 mapping 字段用于指明这个映射怎么使用，和至少一个参数（例如 match）来定义这个模板适用于哪个字段。
模板按照顺序来检测，第一个匹配的模板会被启用。例如，我们给 string 类型字段定义两个模板：
es: 字段名以 _es 结尾需要使用 spanish 分析器。
en: 所有其他字段使用 english 分析器。
我们将 es 模板放在第一位，因为它比匹配所有字符串的 en 模板更特殊一点
```

PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic_templates": [
                { "es": {
                      "match":              "*_es", <1>
                      "match_mapping_type": "string",
                      "mapping": {
                          "type":           "string",
                          "analyzer":       "spanish"
                      }
                }},
                { "en": {
                      "match":              "*", <2>
                      "match_mapping_type": "string",
                      "mapping": {
                          "type":           "string",
                          "analyzer":       "english"
                      }
                }}
            ]
}}}

```
匹配字段名以 _es 结尾的字段. 匹配所有字符串类型字段。
match_mapping_type 允许你限制模板只能使用在特定的类型上，就像由标准动态映射规则检测的一样，（例如 strong 和 long）
match 参数只匹配字段名，path_match 参数则匹配字段在一个对象中的完整路径，所以 address.*.name 规则将匹配一个这样的字段：
```

{
    "address": {
        "city": {
            "name": "New York"
        }
    }
}

```
unmatch 和 path_unmatch 规则将用于排除未被匹配的字段。

##  默认映射 
**通常，一个索引中的所有类型具有共享的字段和设置。用 _default_ 映射来指定公用设置会更加方便，而不是每次创建新的类型时重复操作。**_default 映射像新类型的模板。所有在 _default_ 映射 之后 的类型将包含所有的默认设置，除非在自己的类型映射中明确覆盖这些配置。
例如，我们可以使用 _default_ 映射对所有类型禁用 _all 字段，而只在 blog 字段上开启它：
```

PUT /my_index
{
    "mappings": {
        "_default_": {
            "_all": { "enabled":  false }
        },
        "blog": {
            "_all": { "enabled":  true  }
        }
    }
}

```
**_default_ 映射也是定义索引级别的动态模板的好地方。**

##  重新索引数据 
虽然你可以给索引添加新的类型，或给类型添加新的字段，但是你不能添加新的分析器或修改已有字段。假如你这样做，已被索引的数据会变得不正确而你的搜索也不会正常工作。
**修改在已存在的数据最简单的方法是重新索引：创建一个新配置好的索引，然后将所有的文档从旧的索引复制到新的上。**
_source 字段的一个最大的好处是你已经在 Elasticsearch 中有了完整的文档，你不再需要从数据库中重建你的索引，这样通常会比较慢。
**为了更高效的索引旧索引中的文档，使用【scan-scoll】来批量读取旧索引的文档，然后将通过【bulk API】来将它们推送给新的索引。**
批量重新索引：
你可以在同一时间执行多个重新索引的任务，但是你显然不愿意它们的结果有重叠。所以，可以将重建大索引的任务通过日期或时间戳字段拆分成较小的任务：
GET /old_index/_search?` search_type=scan&scroll=1m `
```

{
    "query": {
        "range": {
            "date": {
                "gte":  "2014-01-01",
                "lt":   "2014-02-01"
            }
        }
    },
    "size":  1000
}

```
假如你继续在旧索引上做修改，你可能想确保新增的文档被加到了新的索引中。这可以通过重新运行重建索引程序来完成，但是记得只要过滤出上次执行后新增的文档就行了。

##  索引别名和零停机时间 
前面提到的重新索引过程中的问题是必须更新你的应用，来使用另一个索引名。索引别名正是用来解决这个问题的！
索引 别名 就像一个快捷方式或软连接，可以指向一个或多个索引，也可以给任何需要索引名的 API 使用。**别名带给我们极大的灵活性，允许我们做到：**
  * 在一个运行的集群上无缝的从一个索引切换到另一个
  * 给多个索引分类（例如，last_three_months）
  * 给索引的一个子集创建 视图
我们以后会讨论更多别名的使用场景。现在我们将介绍用它们怎么在零停机时间内从旧的索引切换到新的索引。
**这里有两种管理别名的途径：_alias 用于单个操作，_aliases 用于原子化多个操作。**
在这一章中，我们假设你的应用采用一个叫 my_index 的索引。**而事实上，my_index 是一个指向当前真实索引的别名。**真实的索引名将包含一个版本号：my_index_v1, my_index_v2 等等。
开始，我们创建一个索引 my_index_v1，然后将别名 my_index 指向它：
PUT /my_index_v1 <1>
` PUT ` /my_index_v1/` _alias `/my_index <2>
创建索引 my_index_v1。 将别名 my_index 指向 my_index_v1。
**你可以检测这个别名指向哪个索引：**
GET /` */_alias `/my_index
或哪些别名指向这个索引：
GET /my_index_v1` /_alias/* `
两者都将返回下列值：
```

{
    "my_index_v1" : {
        "aliases" : {
            "my_index" : { }
        }
    }
}

```
然后，我们决定修改索引中一个字段的映射。当然我们不能修改现存的映射，索引我们需要重新索引数据。首先，我们创建有新的映射的索引 my_index_v2。
PUT /my_index_v2
然后我们从将数据从 my_index_v1 迁移到 my_index_v2，下面的过程在【重新索引】中描述过了。**一旦我们认为数据已经被正确的索引了，我们就将别名指向新的索引。**
**别名可以指向多个索引，所以我们需要在新索引中添加别名的同时从旧索引中删除它。这个操作需要原子化，所以我们需要用 _aliases 操作：**
` POST /_aliases `
```

{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}

```
这样，你的应用就从旧索引迁移到了新的，而没有停机时间。

提示：
即使你认为现在的索引设计已经是完美的了，当你的应用在生产环境使用时，还是有可能在今后有一些改变的。
所以请做好准备：**在应用中使用别名而不是索引。然后你就可以在任何时候重建索引。别名的开销很小，应当广泛使用。**
