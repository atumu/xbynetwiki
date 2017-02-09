title: luceneinaction_第2版_学习笔记9 

#  LuceneInAction2rd学习笔记之分页 
两种方案
通过关键字的检索，当lucene返回多条记录的时候，往往一个页面是无法容纳所有检索结果的，这自然而然就该分页了。我这里给出两种方案，这两种方法我都是用过。
第一种方法，就是讲检索结果全部封装在一个Collection中，例如List中，将这个结果传到前台，如jsp页面。然后在这个list中进行分页显示；
第二种方法，是使用lucene自带的分页工具public TopDocs topDocs(int start,int howMany)。
我认为，第一种方法不涉及二次查询，这样的话就避免了在查询上的浪费。但是当检索的结果数据量很大，这样一次性传输这么多数据到客户端，而用户检索后得到的结果往往只会查看第一页的内容，很少去查看第二页、第三页以及后面的内容，所以一次性将全部结果传到前台，这样的浪费是很大的。
 **第二种方法，虽然每次翻页都意味着一次查询，表面上浪费了资源，但是由于lucene的高效，这样的浪费对整个系统的影响是微乎其微的，但是这个方法避免了方法一中的缺陷。**
分页实现
```

/** 
     * 对搜索返回的前n条结果进行分页显示 
     * @param keyWord       查询关键词 
     * @param pageSize      每页显示记录数 
     * @param currentPage   当前页  
     */  
    public void paginationQuery(String keyWord,int pageSize,int currentPage) throws ParseException, CorruptIndexException, IOException {  
        String[] fields = {"title","content"};  
        QueryParser queryParser = new MultiFieldQueryParser(Version.LUCENE_36,fields,analyzer);  
        Query query = queryParser.parse(keyWord);  
           
        IndexReader indexReader  = IndexReader.open(directory);  
        IndexSearcher indexSearcher = new IndexSearcher(indexReader);  
           
        //TopDocs 搜索返回的结果  
        TopDocs topDocs = indexSearcher.search(query, 100);//只返回前100条记录  
        int totalCount = topDocs.totalHits; // 搜索结果总数量  
        ScoreDoc[] scoreDocs = topDocs.scoreDocs; // 搜索返回的结果集合  
           
        //查询起始记录位置  
        int begin = pageSize * (currentPage - 1) ;  
        //查询终止记录位置  
        int end = Math.min(begin + pageSize, scoreDocs.length);  
           
        //进行分页查询  
        for(int i=begin;i<end;i++) {  
            int docID = scoreDocs[i].doc;  
            Document doc = indexSearcher.doc(docID);  
            int id = NumericUtils.prefixCodedToInt(doc.get("id"));  
            String title = doc.get("title");  
            System.out.println("id is : "+id);  
            System.out.println("title is : "+title);  
        }     
    }  

```