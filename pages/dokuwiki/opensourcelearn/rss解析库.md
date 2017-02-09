title: rss解析库 

#  ROME-RSS解析库 

一:什么是RSS
 RSS(really simple syndication) :网页内容聚合器。RSS的格式是XML。必须符合XML 1.0规范。
 RSS的作用：订阅BLOG,订阅新闻
二:RSS的历史版本:
 http://blogs.law.harvard.edu/tech/rssVersionHistory
 RSS的版本有很多个，0.90、0.91、0.92、0.93、0.94、1.0 和 2.0。与RSS相对的还有ATOM。
 国内主要是RSS2.0,国外主要用ATOM0.3.
 由于RSS出现2派，导致混乱场面。其中RSS2.0规范由哈佛大学定义并锁定。
 地址:http://blogs.law.harvard.edu/tech/rss
三：RSS 文件形式
1:例子：
```

<?xml version="1.0" encoding="utf-8" ?>
<!-- 声明当前文件为xml文档【必】 -->
<rss version="2.0">
	<!-- 声明当前文件内容为rss格式文件,属性version（必须）指定当前rss版本【必】 -->
	<channel>
		<!-- 固有节点【必】 -->
		<title>新闻中心-国内焦点新闻</title>
		<!-- 对网站和当前RSS 文件的简短描述【必】 -->
		<image>
			<!-- 为当前RSS添加图片 -->
			<title>新闻中心-国内焦点</title>
			<!-- 图片标题对图片的简单描述 -->
			<link>http://news.sina.com.cn/china</link>
			<!-- 网站链接地址 -->
			<url>http://image2.sina.com.cn/dy/gn/in10.jpg</url>
			<!-- 图片的链接地址 -->
		</image>
		<description>国内焦点新闻列表</description>
		-<!-- 对当前RSS文件的描述【必】 -->
		<link>http://news.sina.com.cn/china/index.shtml</link>
		<!-- 网站主页链接【必】 -->
		<language>zh-cn</language>
		<!-- 当前RSS使用的语言 -->
		<generator>WWW.SINA.COM.CN</generator>
		<!-- 当RSS文件为自动创建时多存在此节点（RSS文件由什么创建） -->
		<ttl>5</ttl>
		<!-- (ttl = time to live) 在刷新前当前RSS在cache中可以保存多长时间（分钟） -->
		<copyright>Copyright 1996 - 2005 SINA Inc. All Rights Reserved</copyright>
		<!-- 声明版权 -->
		<pubDate>Wed, 26 Apr 2006 01:45:05 GMT</pubDate>
		<!-- 当前RSS最后发布的时间 -->
		<category />
		<!-- 声明当前RSS内容的种类 -->
		<item>
			<!-- 一条信息 -->
			<title>最高检：严惩公务员利用审批等权力索贿受贿</title>
			<!-- 新闻标题【必】 -->
			<link>http://news.sina.com.cn/c/l/2006-04-26/08029720281.shtml</link>
			<!-- 新闻链接【必】 -->
			<author>WWW.SINA.COM.CN</author>
			<!-- 新闻作者 -->
			<guid>http://news.sina.com.cn/c/l/2006-04-26/08029720281.shtml</guid>
			<!-- guid>GUID=Globally Unique Identifier 为当前新闻指定一个全球唯一标示 -->
			<category>国内焦点新闻</category>
			<!-- 新闻种类 -->
			<pubDate>Wed, 26 Apr 2006 00:02:53 GMT</pubDate>
			<!-- 新闻最后发布时间 -->
			<comments />
			<!-- 新闻注释 -->
			<description>新华网沈阳4月25日电 (记者 杨维汉、范春生)
				最高人民检察院常务副检察长张耕说，对于国家公务员在商业活动中利用职权谋取非法利益、索贿受贿的案件，必须发现一起，坚决查处一起。特别是对国家公务员利用行政审批权、行政执法权和司法权执法犯法、贪赃枉法、索贿受贿，构成犯....</description>
			<!-- 新闻的简单描述【必】 -->
		</item>
	</channel>
</rss>

```
 2:RSS文件由一个 <channel> 元素及其子元素组成。除了频道内容本身之外，<channel> 
 还以项的形式包含表示频道元数据的元素 —— 比如 <title>、<link> 和 <description>。
 项通常是频道的主要部分，包含经常变化的内容。
3:频道(channel)用<channel>表示
 频道一般有三个元素，提供关于频道本身的信息：
 <title>：频道或提要的名称。 
 <link>：与该频道关联的 Web 站点或者站点区域的 URL。 
 <description>：简要介绍该频道是做什么的。 
 许多频道子元素都是可选的。常用的 <image> 元素包含三个必需的子元素：
 <url>：表示该频道的 GIF、JPEG 或 PNG 图像的 URL。 
 <title>：图象的描述。当频道以 HTML 呈现时，用作 HTML <image> 标签的 ALT 属性。 
 <link>：站点的 URL。如果频道以 HTML 呈现，该图像作为到这个站点的链接。 
 <image> 还有三个可选的子元素：
 <width>：数字，表示图象的像素宽度，最大值是 188，默认值为 88。 
 <height>：数字，表示图象的像素高度。最大值是 400，默认值为 31。 
 <description>：包含文本，在呈现时可以作为围绕着该图像形成的链接元素的 title 属性。 
 此外还可以使用许多其他可选的频道元素。多数都是不言自明的：
 <language>：en-us 
 <copyright>：Copyright 2003, James Lewin 
 <managingEditor>：dan@spam_me.com (Dan Deletekey) 
 <webMaster>：dan@spam_me.com (Dan Deletekey) 
 <pubDate>：Sat, 15 Nov 2003 0:00:01 GMT 
 <lastBuildDate>：Sat, 15 Nov 2003 0:00:01 GMT 
 <category>：ebusiness 
 <generator>：Your CMS 2.0 
 <docs>：http://blogs.law.harvard.edu/tech/rss 
 <cloud>：允许进程注册为“cloud”，频道更新时通知它，为 RSS 提要实现了一种轻量级的发布-订阅协议。 
 <ttl>：存活时间 是一个数字，表示提要在刷新之前缓冲的分钟数。 
 <rating>：关于该频道的 PICS 评价。 
 <textInput>：定义可与频道一起显示的输入框。 
 <skipHours>：告诉聚集器哪些小时的更新可以忽略。 
 <skipDays>：告诉聚集器那一天的更新可以忽略。 
4:摘要(feed)用<item>表示,<item>的格式如下：
 每个摘要通常包含三个元素：
 <title>：这是项的名称，在标准应用中被转换成 HTML 中的标题。 
 <link>：这是该项的 URL。title 通常作为一个链接，指向包含在 <link> 元素中的 URL。 
 <description>：通常作为 link 中所指向的 URL 的摘要或者补充。 
 所有的元素都是可选的，但是一个项至少要么 包含一个 <title>，要么包含一个 <description>。
 项还有其他一些可选的元素：
 <author>：作者的 e-mail 地址。 
 <category>：支持有组织的记录。 
 <comments>：关于项的注释页的 URL。 
 <enclosure>：支持和该项有关的媒体对象。 
 <guid>：唯一与该项联系在一起的永久性链接。 
 <pubDate>：该项是什么时候发布的。 
 <source>：该项来自哪个 RSS 频道.
四:主流java rss lib及其评测:
Rome:http://rometools.github.io/rome/ 本文主要介绍Rome
rssutils
rsslibj
sandler

#  Rome介绍及使用 
你可以使用Rome解析或者创建RSS Feed.本文只讲解解析部分。
官网：http://rometools.github.io/rome/
安装：
依赖jdom
所以需要安装jdom.jar,rome.jar
<note important>` 注意：对于CSDN博客的RSS订阅，以及相关的限制类订阅，需要制定User-Agent头部。否则会禁止访问。并返回403 `</note>。如下形式：
```

String rss="http://blog.csdn.net/xbynet/rss/list";			
URL url = new URL(rss);
URLConnection conn=url.openConnection();
		conn.addRequestProperty(
				"User-Agent",
				"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36");

```
基本使用：
```

URL feedUrl = new URL("file:blogging-roller.rss");
SyndFeedInput input = new SyndFeedInput();
SyndFeed feed = input.build(new InputStreamReader(feedUrl.openStream()));
/**或者
*SyndFeedInput input = new SyndFeedInput();
*SyndFeed feed = input.build(new XmlReader(feedUrl));
*/
// 得到Rss新闻中子项列表
List entries = feed.getEntries();
// 循环得到每个子项信息
			for (int i = 0; i < entries.size(); i++) {
				SyndEntry entry = (SyndEntry) entries.get(i);
				// 标题、连接地址、标题简介、时间是一个Rss源项最基本的组成部分
				System.out.println("标题：" + entry.getTitle());
				System.out.println("连接地址：" + entry.getLink());
				SyndContent description = entry.getDescription();
				System.out.println("标题简介：" + description.getValue());
				System.out.println("发布时间：" + entry.getPublishedDate());
				// 以下是Rss源可先的几个部分
				System.out.println("标题的作者：" + entry.getAuthor());
				// 此标题所属的范畴
				List categoryList = entry.getCategories();
			}
...

```
示例：
```

import java.net.URL;
import java.util.List;

import com.sun.syndication.feed.synd.SyndCategory;
import com.sun.syndication.feed.synd.SyndContent;
import com.sun.syndication.feed.synd.SyndEnclosure;
import com.sun.syndication.feed.synd.SyndEntry;
import com.sun.syndication.feed.synd.SyndFeed;
import com.sun.syndication.io.SyndFeedInput;
import com.sun.syndication.io.XmlReader;

public class TestRss {
	public static void main(String[] args) {
		TestRss test = new TestRss();
		test.parseRss();
	}

	public void parseRss() {
		// String rss =
		// "http://news.baidu.com/n?cmd=1&class=civilnews&tn=rss&sub=0]http://news.baidu.com/n?cmd=1&class=civilnews&tn=rss&sub=0";
		String rss = "http://localhost:8800/feed.php?mode=list&ns=java";
		try {
			URL url = new URL(rss);
			// 读取Rss源
			XmlReader reader = new XmlReader(url);
			System.out.println("Rss源的编码格式为：" + reader.getEncoding());
			SyndFeedInput input = new SyndFeedInput();
			// 得到SyndFeed对象，即得到Rss源里的所有信息
			SyndFeed feed = input.build(reader);
			// System.out.println(feed);
			// 得到Rss新闻中子项列表
			List entries = feed.getEntries();
			// 循环得到每个子项信息
			for (int i = 0; i < entries.size(); i++) {
				SyndEntry entry = (SyndEntry) entries.get(i);
				// 标题、连接地址、标题简介、时间是一个Rss源项最基本的组成部分
				System.out.println("标题：" + entry.getTitle());
				System.out.println("连接地址：" + entry.getLink());
				SyndContent description = entry.getDescription();
				System.out.println("标题简介：" + description.getValue());
				System.out.println("发布时间：" + entry.getPublishedDate());
				// 以下是Rss源可先的几个部分
				System.out.println("标题的作者：" + entry.getAuthor());
				// 此标题所属的范畴
				List categoryList = entry.getCategories();
				if (categoryList != null) {
					for (int m = 0; m < categoryList.size(); m++) {
						SyndCategory category = (SyndCategory) categoryList
								.get(m);
						System.out.println("此标题所属的范畴：" + category.getName());
					}
				}
				// 得到流媒体播放文件的信息列表
				List enclosureList = entry.getEnclosures();
				if (enclosureList != null) {
					for (int n = 0; n < enclosureList.size(); n++) {
						SyndEnclosure enclosure = (SyndEnclosure) enclosureList
								.get(n);
						System.out.println("流媒体播放文件：" + entry.getEnclosures());
					}
				}
				System.out.println();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}


```
参考：
http://www.360doc.com/content/10/1224/09/1007797_80869844.shtml
http://blog.csdn.net/earbao/article/details/33741185
http://rometools.github.io/rome/