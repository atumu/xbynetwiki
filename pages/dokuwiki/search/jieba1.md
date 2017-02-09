title: jieba1 

#  jieba中文分词 
官网：https://github.com/fxsjy/jieba/
分词函数
  * jieba.cut 方法接受三个输入参数: 需要分词的字符串；cut_all 参数用来控制是否采用全模式；HMM 参数用来控制是否使用 HMM 模型
  * jieba.cut_for_search 方法接受两个参数：需要分词的字符串；是否使用 HMM 模型。该方法适合用于搜索引擎构建倒排索引的分词，粒度比较细
  * 待分词的字符串可以是 unicode 或 UTF-8 字符串、GBK 字符串。注意：不建议直接输入 GBK 字符串，可能无法预料地错误解码成 UTF-8
  * jieba.cut 以及 jieba.cut_for_search 返回的结构都是一个可迭代的 generator，可以使用 for 循环来获得分词后得到的每一个词语(unicode)，或者用
  * jieba.lcut 以及 jieba.lcut_for_search 直接返回 list
  * jieba.Tokenizer(dictionary=DEFAULT_DICT) 新建自定义分词器，可用于同时使用不同词典。jieba.dt 为默认分词器，所有全局分词相关函数都是该分词器的映射。

```

# encoding=utf-8
import jieba

seg_list = jieba.cut("我来到北京清华大学", cut_all=True)
print("Full Mode: " + "/ ".join(seg_list))  # 全模式

seg_list = jieba.cut("我来到北京清华大学", cut_all=False)
print("Default Mode: " + "/ ".join(seg_list))  # 精确模式

seg_list = jieba.cut("他来到了网易杭研大厦")  # 默认是精确模式
print(", ".join(seg_list))

seg_list = jieba.cut_for_search("小明硕士毕业于中国科学院计算所，后在日本京都大学深造")  # 搜索引擎模式
print(", ".join(seg_list))

```

##  2. 添加自定义词典 
载入词典
开发者可以指定自己自定义的词典，以便包含 jieba 词库里没有的词。虽然 jieba 有新词识别能力，但是自行添加新词可以保证更高的正确率
用法： ` jieba.load_userdict(file_name) ` # file_name 为文件类对象或自定义词典的路径
词典格式和 dict.txt 一样，一个词占一行；每一行分三部分：词语、词频（可省略）、词性（可省略），用空格隔开，顺序不可颠倒。file_name 若为路径或二进制方式打开的文件，则文件必须为 UTF-8 编码。
词频省略时使用自动计算的能保证分出该词的词频。

更改分词器（默认为 jieba.dt）的 tmp_dir 和 cache_file 属性，可分别指定缓存文件所在的文件夹及其文件名，用于受限的文件系统。
范例：
自定义词典：https://github.com/fxsjy/jieba/blob/master/test/userdict.txt
用法示例：https://github.com/fxsjy/jieba/blob/master/test/test_userdict.py

调整词典
使用 add_word(word, freq=None, tag=None) 和 del_word(word) 可在程序中动态修改词典。
使用 suggest_freq(segment, tune=True) 可调节单个词语的词频，使其能（或不能）被分出来。
注意：自动计算的词频在使用 HMM 新词发现功能时可能无效。

"通过用户自定义词典来增强歧义纠错能力" --- https://github.com/fxsjy/jieba/issues/14

##  Tokenize：返回词语在原文的起止位置 
注意，输入参数只接受 unicode
默认模式
```

result = jieba.tokenize(u'永和服装饰品有限公司')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
word 永和                start: 0                end:2
word 服装                start: 2                end:4
word 饰品                start: 4                end:6
word 有限公司            start: 6                end:10

```

搜索模式
```

result = jieba.tokenize(u'永和服装饰品有限公司', mode='search')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
word 永和                start: 0                end:2
word 服装                start: 2                end:4
word 饰品                start: 4                end:6
word 有限                start: 6                end:8
word 公司                start: 8                end:10
word 有限公司            start: 6                end:10

```
##  7. ChineseAnalyzer for Whoosh 搜索引擎 
引用：** from jieba.analyse import ChineseAnalyzer**
用法示例：https://github.com/fxsjy/jieba/blob/master/test/test_whoosh.py

##  8. 命令行分词 
使用示例：python -m jieba news.txt > cut_result.txt
命令行选项（翻译）：
使用: python -m jieba [options] filename
结巴命令行界面。
固定参数:
filename              输入文件
可选参数:
```

  -h, --help            显示此帮助信息并退出
  -d [DELIM], --delimiter [DELIM]
                        使用 DELIM 分隔词语，而不是用默认的' / '。
                        若不指定 DELIM，则使用一个空格分隔。
  -p [DELIM], --pos [DELIM]
                        启用词性标注；如果指定 DELIM，词语和词性之间
                        用它分隔，否则用 _ 分隔
  -D DICT, --dict DICT  使用 DICT 代替默认词典
  -u USER_DICT, --user-dict USER_DICT
                        使用 USER_DICT 作为附加词典，与默认词典或自定义词典配合使用
  -a, --cut-all         全模式分词（不支持词性标注）
  -n, --no-hmm          不使用隐含马尔可夫模型
  -q, --quiet           不输出载入信息到 STDERR
  -V, --version         显示版本信息并退出

```

如果没有指定文件名，则使用标准输入。

