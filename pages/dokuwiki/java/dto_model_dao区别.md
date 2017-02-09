title: dto_model_dao区别 

#  DTO、Model、DAO区别 
平时也不是很注意他们的区别，尤其是Model与DTO的区别都没好好想过。这次记录下来。其实理解这些区别对于开发而言还是很重要的。

###  领域驱动设计的分层 
领域驱动设计将软件系统分为四层：基础结构层、领域层、应用层和表现层。数据访问层DAO已经被移到基础结构层了。
PO：persistant object持久对象
BO：business object业务对象
**DTO** ：Data Transfer Object数据传输对象.**DTO通常用于不同层（UI层、服务层或者域模型层）直接的数据传输，以隔离不同层，降低层间耦合**.
POJO ：plain ordinary java object 简单java对象
一个POJO持久化以后就是PO
直接用它传递、传递过程中就是DTO
**DAO**：data access object数据访问对象
这个大家最熟悉，和上面几个O区别最大，基本没有互相转化的可能性和必要.
**DAO主要用来封装对数据库的访问**。通过它可以把POJO持久化为PO，用PO组装出来VO、DTO
**Domain Model**:领域模型对象，封装数据和业务逻辑

<WRAP center round tip 60%>
在使用领域模型的情况下，如果没有DTO层，那么Model是一定会完整地暴露给Client端。所以注意对DTO的设计。
</WRAP>

![](/data/dokuwiki/java/pasted/20150819-085918.png)![](/data/dokuwiki/java/pasted/20150819-085952.png)
##  我们为什么需要DTO(数据传输对象) 
**DTO(Data Transfer Object)即数据传输对象**。之前不明白有些框架中为什么要专门定义DTO来绑定表现层中的数据，为什么不能直接用实体模型呢，有了DTO同时还要维护DTO与Model之间的映射关系，多麻烦。
具体可以参考这篇文章：http://www.cnblogs.com/daxnet/archive/2010/07/07/1772584.html
**表现层与应用层之间是通过数据传输对象（DTO）进行交互的**
数据传输对象DTO是**没有行为的POJO对象**，它 的目的**只是为了对领域对象进行数据封装**，实现层与层之间的**数据传递**。
为何不能直接将领域对象用于 数据传递？
因为领域对象更注重领域，**而DTO更注重数据。**不仅如此，由于“富领域模型”的特点，这样 做会直接将领域对象的行为暴露给表现层。
DTO层的作用是为了隔离Domain Model：**让DoMain Model的改动不会直接影响到UI**；保持Domain Model的安全,**不暴露业务逻辑**。

需要了解的是，数据传输对象DTO本身并不是业务对象。数据传输对象是根据UI的需求进行设计的，而不 是根据领域对象进行设计的。比如，Customer领域对象可能会包含一些诸如FirstName, LastName, Email, Address等信息。但如果UI上不打算显示Address的信息，那么CustomerDTO中也无需包含这个 Address的数据

简单来说**Model面向业务**，我们是通过业务来定义Model的。**而DTO是面向界面UI**，是通过UI的需求来定义的。**通过DTO我们实现了表现层与Model之间的解耦**，表现层不引用Model，如果开发过程中我们的模型改变了，而界面没变，我们就只需要改Model而不需要去改表现层中的东西。

参考：
http://www.cnblogs.com/Gyoung/archive/2013/03/23/2977233.html
http://www.cnblogs.com/daxnet/archive/2010/07/07/1772584.html
http://www.blogjava.net/vip01/archive/2007/01/08/92430.html