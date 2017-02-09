title: phplearn11 

#  PHP学习之JSON与XML处理 
##  XML处理 
内建的 Expat 解析器使在 PHP 中处理 XML 文档成为可能。
内建的 DOM 解析器使在 PHP 中处理 XML 文档成为可能。
SimpleXML 处理最普通的 XML 任务，其余的任务则交由其它扩展。
有两种基本的 XML 解析器类型：
  * 基于树的解析器：这种解析器把 XML 文档转换为树型结构。它分析整篇文档，并提供了 API 来访问树种的元素，例如文档对象模型 (DOM)。
  * 基于事件的解析器：将 XML 文档视为一系列的事件。当某个具体的事件发生时，解析器会调用函数来处理。
**SimpleXML 是 PHP 5 中的新特性。在了解 XML 文档 layout 的情况下，它是一种取得元素属性和文本的便利途径。**
与 DOM 或 Expat 解析器相比，SimpleXML 仅仅用几行代码就可以从元素中读取文本数据。
SimpleXML 可把 XML 文档转换为对象，比如：
  * 元素 - 被转换为 SimpleXMLElement 对象的单一属性。当同一级别上存在多个元素时，它们会被置于数组中。
  * 属性 - 通过使用关联数组进行访问，其中的下标对应属性名称。
  * 元素数据 - 来自元素的文本数据被转换为字符串。如果一个元素拥有多个文本节点，则按照它们被找到的顺序进行排列。
当执行类似下列的基础任务时，SimpleXML 使用起来非常快捷：
  * 读取 XML 文件
  * 从 XML 字符串中提取数据
  * 编辑文本节点或属性
不过，在处理高级 XML 时，比如命名空间，最好使用 Expat 解析器或 XML DOM。

##  使用 SimpleXML 
下面是 XML 文件：
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<note>
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note>

```
我们打算从上面的 XML 文件输出元素的名称和数据。
这是需要做的事情：
  * 加载 XML 文件
  * 取得第一个元素的名称
  * 使用 children() 函数创建在每个子节点上触发的循环
  * 输出每个子节点的元素名称和数据
```

<?php
$xml = simplexml_load_file("test.xml");
echo $xml->getName() . "<br />";
foreach($xml->children() as $child)
  {
  echo $child->getName() . ": " . $child . "<br />";
  }
?>

```
以上代码的输出：
note
to: George
from: John
heading: Reminder
body: Don't forget the meeting!
