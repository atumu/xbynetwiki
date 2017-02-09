title: lucene4和solr4下一代搜索和分析 

#  使用Lucene4和Solr4实现下一代搜索和分析 
Apache Lucene™ 和 Solr™ 是强大的开源搜索技术，使组织能够轻松地显著增强数据访问。借助 4.x 版的 Lucene 和 Solr，向数据驱动应用程序中**添加可扩展**的搜索功能变得比以往更加轻松。Lucene 和 Solr 提交者 Grant Ingersoll 介绍了与相关性、分布式搜索和分面 (facet) 相关的最新 Lucene 和 Solr 功能。本文将学习如何利用这些功能**构建快速、高效、可扩展**的下一代数据驱动应用程序。

这些年来，Lucene 和 Solr 将自身建设成了一项坚不可摧的技术（Lucene 作为 Java™ API 的基础，Solr 作为搜索服务.）
多年来，大部分人对 Lucene 和 Solr 的使用主要集中在基于文本的搜索上。与此同时，**新的、有趣的大数据趋势以及对分布式计算和大规模分析的全新（重新）关注正在兴起。大数据常常还需要实时的、大规模的信息访问。**鉴于这种转变，Lucene 和 Solr 社区发现自己走到了十字路口：Lucene 的核心支柱开始在大数据应用程序的压力下呈现老态，比如对 Twittersphere 的所有消息建立索引（参见 参考资料）。此外，Solr 在原生分布式索引支持上的匮乏，使得 IT 组织越来越难以富有成本效益的方式扩展他们的搜索基础架构。

该社区开始全面改革 Lucene 和 Solr 支柱（并在某些情况下改革公共 API）。我们的关注点已转向实现**轻松的可伸缩性、近实时的索引和搜索**，以及许多 NoSQL 功能 — 同时利用核心引擎功能。这次全面改革的结晶是 Apache** Lucene4.x 和 Solr 4.x 版本。这些版本首当其冲的目标是解决下一代、大规模、数据驱动的访问和分析问题。**


不过搜索服务器还有一个叫做：**ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。**目前项目使用的是这个ElasticSearch
参考https://www.ibm.com/developerworks/cn/java/j-solr-lucene/