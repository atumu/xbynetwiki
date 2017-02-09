title: elasticsearch入门5 

#  Elasticsearch入门之映射和分析 
映射(mapping)机制用于进行字段类型确认，将每个字段匹配为一种确定的数据类型(string, number, booleans, date等)。
分析(analysis)机制用于进行**全文文本(Full Text)**的分词，以建立供搜索用的反向索引。

##  数据类型映射索引差异 

当在索引中处理数据时，我们注意到一些奇怪的事。有些东西似乎被破坏了：
在索引中有12个tweets，只有一个包含日期2014-09-15，但是我们看看下面查询中的total hits。

GET /_search?q=2014              # 12 个结果
GET /_search?q=2014-09-15        # 还是 12 个结果 !
GET /_search?q=date:2014-09-15   # 1  一个结果
GET /_search?q=date:2014         # 0  个结果 !
为什么全日期的查询返回所有的tweets，而针对date字段进行年度查询却什么都不返回？ 为什么我们的结果因查询_all字段(译者注：默认所有字段中进行查询)或date字段而变得不同？

想必是因为我们的数据在_all字段的索引方式和在date字段的索引方式不同而导致。
让我们**看看Elasticsearch在对gb索引中的tweet类型进行_mapping_**(也称之为_模式定义_[注：此词有待重新定义(schema definition)])后是如何解读我们的文档结构：
GET /gb/` _mapping `/tweet
返回：
```

{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}

```
**Elasticsearch为对字段类型进行猜测，动态生成了字段和类型的映射关系。**返回的信息显示了**date字段被识别为date类型。_all因为是默认字段所以没有在此显示，不过我们知道它是string类型。**
**date类型的字段和string类型的字段的索引方式是不同的，因此导致查询结果的不同，这并不会让我们觉得惊讶。**
你会期望每一种核心数据类型(strings, numbers, booleans及dates)**以不同的方式进行索引**，而这点也是现实：在Elasticsearch中他们是被区别对待的。
**但是更大的区别在于_确切值_(exact values)(比如string类型)及_全文文本_(full text)之间。**
这两者的区别才真的很重要 - ` 这是区分搜索引擎和其他数据库的根本差异。 `

##  确切值(Exact values) vs. 全文文本(Full text) 
Elasticsearch中的数据可以大致分为两种类型：**确切值 及 全文文本**。

确切值是确定的，正如它的名字一样。**比如一个date或用户ID**，也可以包含更多的字符串比如username或email地址。确切值"Foo"和"foo"就并不相同。确切值2014和2014-09-15也不相同。
全文文本，从另一个角度来说是文本化的数据(常常以人类的语言书写)，比如一片推文(Twitter的文章)或邮件正文。
全文文本常常被称为非结构化数据，其实是一种用词不当的称谓，实际上自然语言是高度结构化的。
问题是自然语言的语法规则是如此的复杂，计算机难以正确解析。例如这个句子：
May is fun but June bores me.
到底是说的月份还是人呢？
**确切值是很容易查询的，因为结果是二进制的 -- 要么匹配，要么不匹配。**下面的查询很容易以SQL表达：
```

WHERE name    = "John Smith"
  AND user_id = 2
  AND date    > "2014-09-15"

```
而对于全文数据的查询来说，却有些微妙。我们不会去询问这篇文档是否匹配查询要求？。** 但是，我们会询问这篇文档和查询的匹配程度如何？**。换句话说，对于查询条件，这篇文档的_相关性_有多高？
我们很少确切的匹配整个全文文本。我们想在全文中查询*包含*查询文本的部分。不仅如此，我们还期望搜索引擎能理解我们的*意图*：
一个针对"UK"的查询将返回涉及"United Kingdom"的文档
一个针对"jump"的查询同时能够匹配"jumped"， "jumps"， "jumping"甚至"leap"
"johnny walker"也能匹配"Johnnie Walker"， "johnnie depp"及"Johnny Depp"
"fox news hunting"能返回有关hunting on Fox News的故事，而"fox hunting news"也能返回关于fox hunting的新闻故事。
为了方便在全文文本字段中进行这些类型的查询，Elasticsearch首先对文本**分析(analyzes)**，然后使用结果建立一个**倒排索引**。我们将在以下两个章节讨论倒排索引及分析过程。

##  指定分析器 
当Elasticsearch在你的文档中探测到一个新的字符串字段，它将自动设置它为全文string字段并用` standard分析器 `分析。
也许你想使用一个更适合这个数据的语言分析器。或者，你只想把字符串字段当作一个普通的字段——不做任何分析，只存储确切值，就像字符串类型的用户ID或者内部状态字段或者标签。
` 为了达到这种效果，我们必须通过**映射(mapping)**人工设置这些字段。 `

##  映射 
索引中每个文档都有一个**类型(type)**。 每个类型拥有自己的**映射(mapping)**或者**模式定义(schema definition)**。
一个映射定义了字段类型，每个字段的数据类型，以及字段被Elasticsearch处理的方式。映射还用于设置关联到类型上的元数据。
在《映射》章节我们将探讨映射的细节。这节我们只是带你入门。
核心简单字段类型
**Elasticsearch支持以下简单字段类型：**
类型	表示的数据类型
  * String	string
  * Whole number	byte, short, integer, long
  * Floating point	float, double
  * Boolean	boolean
  * Date	date
当你索引一个包含新字段的文档——一个之前没有的字段——Elasticsearch将**使用动态映射猜测字段类型，这类型来自于JSON的基本数据类型**，使用以下规则：
JSON type	Field type
Boolean: true or false	"boolean"
Whole number: 123	"long"
Floating point: 123.45	"double"
String, valid date: "2014-09-15"	"date"
String: "foo bar"	"string"
这意味着，**如果你索引一个带引号的数字——"123"，它将被映射为"string"类型，而不是"long"类型。**然而，如果字段已经被映射为"long"类型，Elasticsearch将尝试转换字符串为long，并在转换失败时会抛出异常。

###  查看映射 
我们可以使用_mapping后缀来查看Elasticsearch中的映射。在本章开始我们已经找到索引gb类型tweet中的映射：
GET /gb/` _mapping `/tweet
这展示给了我们字段的映射（叫做**属性(properties)**），这些映射是Elasticsearch在创建索引时动态生成的：
```

{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}

```
小提示
错误的映射，例如把age字段映射为string类型而不是integer类型，会造成查询结果混乱。
要检查映射类型，而不是假设它是正确的！

###  自定义字段映射 
映射中最重要的字段参数是type。除了string类型的字段，你可能很少需要映射其他的type：
```

{
    "number_of_clicks": {
        "type": "integer"
    }
}

```
**string类型的字段，默认的，考虑到包含全文本，它们的值在索引前要经过分析器分析，并且在全文搜索此字段前要把查询语句做分析处理。**
对于string字段，` 两个最重要的映射参数是index和analyer `。
index
**index参数控制字符串以何种方式被索引。**它包含以下三个值当中的一个：
值	解释
  * analyzed	首先分析这个字符串，然后索引。换言之，以全文形式索引此字段。
  * not_analyzed	索引这个字段，使之可以被搜索，但是索引内容和指定值一样。不分析此字段。
  * no	不索引这个字段。这个字段不能为搜索到。

` string类型字段默认值是analyzed。 `如果我们想映射字段为确切值，我们需要设置它为not_analyzed：
```

{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}

```
**其他简单类型——long、double、date等等——也接受index参数，但相应的值只能是no和not_analyzed，它们的值不能被分析。**

###  分析 
对于analyzed类型的字符串字段，使用` analyzer参数 `来指定哪一种分析器将在搜索和索引的时候使用。**默认的，Elasticsearch使用standard分析器**，但是你可以通过指定一个内建的分析器来更改它，例如whitespace、simple或english。
```

{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}

```
在《自定义分析器》章节我们将告诉你如何定义和使用自定义的分析器。

###  更新映射 
你可以在第一次创建索引的时候指定映射的类型。此外，你也可以晚些时候为新类型添加映射（或者为已有的类型更新映射）。
你可以向已有映射中**增加**字段，但你不能**修改**它。如果一个字段在映射中已经存在，这可能意味着那个字段的数据已经被索引。如果你改变了字段映射，那已经被索引的数据将错误并且不能被正确的搜索到。
我们可以更新一个映射来增加一个新字段，但是不能把已有字段的类型那个从analyzed改到not_analyzed。
为了演示两个指定的映射方法，让我们首先删除索引gb：
DELETE /gb
然后创建一个新索引，指定tweet字段的分析器为english：
```

PUT /gb <1>
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}

```
<1> 这将创建包含mappings的索引，映射在请求体中指定。
再后来，我们决定在tweet的映射中**增加一个新的not_analyzed类型的文本字段，叫做tag，使用_mapping后缀:**
` PUT ` /gb/` _mapping `/tweet
```

{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}

```
注意到我们不再需要列出所有的已经存在的字段，因为我们没法修改他们。我们的新字段已经被合并至存在的那个映射中。

###  测试映射 
你可以通过名字使用analyze API测试字符串字段的映射。对比这两个请求的输出：
GET /gb/` _analyze `?field=tweet
Black cats <1>

GET /gb/_analyze?field=tag
Black-cats <1>
<1> 我们想要分析的文本被放在请求体中。
tweet字段产生两个词，"black"和"cat",tag字段产生单独的一个词"Black-cats"。换言之，我们的映射工作正常。

##  复合核心字段类型 
**除了之前提到的简单的标量类型，JSON还有null值，数组和对象**，所有这些Elasticsearch都支持：

**多值字段**
我们想让tag字段包含多个字段，这非常有可能发生。我们可以**索引一个标签数组来代替单一字符串**：
{ "tag": [ "search", "nosql" ]}
对于数组不需要特殊的映射。任何一个字段可以包含零个、一个或多个值，同样对于全文字段将被分析并产生多个词。
言外之意，这意味着**数组中所有值必须为同一类型**。你不能把日期和字符窜混合。如果你创建一个新字段，这个字段索引了一个数组，Elasticsearch将使用第一个值的类型来确定这个新字段的类型。
当你从Elasticsearch中取回一个文档，任何一个数组的顺序和你索引它们的顺序一致。你取回的_source字段的顺序同样与索引它们的顺序相同。
然而，数组是做为多值字段被**索引**的，它们没有顺序。在搜索阶段你不能指定“第一个值”或者“最后一个值”。倒不如把数组当作一个**值集合(gag of values)**


**空字段**
当然数组可以是空的。这等价于有零个值。**事实上，Lucene没法存放null值，所以一个null值的字段被认为是空字段。**
这四个字段将被识别为空字段而不被索引：
"empty_string":             "",
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]

**多层对象**
我们需要讨论的最后一个自然JSON数据类型是**对象(object)**——在其它语言中叫做hashed、hashmaps、dictionaries 或者 associative arrays.
内部对象(inner objects)经常用于嵌入一个实体或对象里的另一个地方。例如，做在tweet文档中user_name和user_id的替代，我们可以这样写：
```

{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}

```

**内部对象的映射**
Elasticsearch 会动态的检测新对象的字段，并且映射它们为 object 类型，将每个字段加到 properties 字段下
```

{
  "gb": {
    "tweet": { <1>
      "properties": {
        "tweet":            { "type": "string" },
        "user": { <2>
          "type":             "object",
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   { <2>
              "type":         "object",
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}

```

**根对象. 内部对象.**
对user和name字段的映射与tweet类型自己很相似。事实上，type映射只是object映射的一种特殊类型，我们将 object 称为_根对象_。它与其他对象一模一样，除非它有一些特殊的顶层字段，比如 _source, _all 等等。
内部对象是怎样被索引的
**Lucene 并不了解内部对象。 一个 Lucene 文件包含一个键-值对应的扁平表单。 为了让 Elasticsearch 可以有效的索引内部对象，将文件转换为以下格式：**
```

{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}

```
内部栏位可被归类至name，例如"first"。 为了区别两个拥有相同名字的栏位，**我们可以使用完整_路径_，例如"user.name.first" 或甚至类型名称加上路径："tweet.user.name.first"。**
注意： 在以上扁平化文件中，并没有栏位叫作user也没有栏位叫作user.name。 Lucene 只索引阶层或简单的值，而不会索引复杂的资料结构。

**对象-阵列**
内部对象的阵列
最後，一个包含内部对象的阵列如何索引。 我们有个阵列如下所示：
```

{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}

```
此文件会如我们以上所说的被扁平化，但其结果会像如此：
```

{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}

```
{age: 35}与{name: Mary White}之间的关联会消失，因每个多值的栏位会变成一个值集合，而非有序的阵列。 这让我们可以知道：
是否有26岁的追随者？
但我们无法取得准确的资料如：
是否有26岁的追随者**且名字叫Alex Jones？**
**关联内部对象可解决此类问题，我们称之为_嵌套_对象**，我们之後会在嵌套对象中提到它。

