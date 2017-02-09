title: luceneinaction_第2版_学习笔记7 

#  IndexSearcher单实例对于实时搜索 
Lucene版本:3.0
**一般情况下,lucene的IndexSearcher都要写成单实例**,因为每次创建IndexSearcher对象的时候,它都需要把索引文件加载进来,如果访问量比较大,而索引也比较大,那就很容易造成内存溢出!
但是如果仅仅按照一般的单实例来写的话,如果更新了索引,那么在不重启服务的情况下,Searcher对象是搜索不到索引更新后的内容的.如何解决呢,这里给出一个方法!
在这个方法里,建造了一个Factory类,分别管理IndexReader和IndexSearcher的单实例.
```

/** * 工厂类,负责管理IndexReader、IndexSearcher * @author .K' */
public class LuceneFactory {
	private static IndexReader reader = null;
	private static IndexSearcher searcher = null;

	/**
	 * *获得IndexReader对象,判断是否为最新,不是则重新打开 (以下省略了try...catch) *@param file
	 * 索引路径的File对象 *@return IndexReader对象
	 **/
	public static synchronized IndexReader getIndexReader(File file) {
		if (reader == null) {
			reader = IndexReader.open(SimpleFSDirectory.open(file));
		} else {
			if (!reader.isCurrent()) {
				reader = reader.reopen();
			}
		}
		return reader;
	}

	/**
	 * * 获得IndexSearcher对象,判断当前的Searcher中reader是否为最新,如果不是,则重新创建IndexSearcher(
	 * 省略了try...catch) * @param reader * IndexReader对象 * @return IndexSearcher对象
	 */
	public static synchronized IndexSearcher getIndexSearcher(IndexReader reader) {
		if (searcher == null) {
			searcher = new IndexSearcher(reader);
		} else {
			IndexReader r = searcher.getIndexReader();
			if (!r.isCurrent()) {
				searcher = new IndexSearcher(reader);
			}
		}
		return searcher;
	}
}

```
调用工厂类中的方法创建Searcher
IndexReader reader = LuceneFactory.getIndexReader(new File(path));
Searcher searcher = LuceneFactory.getIndexSearcher(reader);
以上就可以解决对于索引无法实时搜索的问题了!

参考：http://www.cnblogs.com/zhwl/p/3483798.html