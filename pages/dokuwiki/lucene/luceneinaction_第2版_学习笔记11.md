title: luceneinaction_第2版_学习笔记11 

#  LuceneInAction2rd学习笔记之多索引搜索 
某些应用需要保持多个分离的Lucene索引。但又需要在搜索过程中能够使**针对这几个索引的所有搜索结果合并输出**。
Lucene提供了两个类用于进行针对多索引进行搜索：
针对单线程的MultiSearcher
针对多线程的ParallelMultiSearcher
##  使用MultiSearch类 
```

public class MultiSearcherTest extends TestCase {
  private IndexSearcher[] searchers;

  public void setUp() throws Exception {
    String[] animals = { "aardvark", "beaver", "coati",
                       "dog", "elephant", "frog", "gila monster",
                       "horse", "iguana", "javelina", "kangaroo",
                       "lemur", "moose", "nematode", "orca",
                       "python", "quokka", "rat", "scorpion",
                       "tarantula", "uromastyx", "vicuna",
                       "walrus", "xiphias", "yak", "zebra"};

    Analyzer analyzer = new WhitespaceAnalyzer();

    Directory aTOmDirectory = new RAMDirectory();     // #1
    Directory nTOzDirectory = new RAMDirectory();     // #1

    IndexWriter aTOmWriter = new IndexWriter(aTOmDirectory,
                                             analyzer,
                                             IndexWriter.MaxFieldLength.UNLIMITED);
    IndexWriter nTOzWriter = new IndexWriter(nTOzDirectory,
                                             analyzer,
                                             IndexWriter.MaxFieldLength.UNLIMITED);
    

    for (int i=animals.length - 1; i >= 0; i--) {
      Document doc = new Document();
      String animal = animals[i];
      doc.add(new Field("animal", animal, Field.Store.YES, Field.Index.NOT_ANALYZED));
      if (animal.charAt(0) < 'n') {
        aTOmWriter.addDocument(doc);                 // #2
      } else {                                       
        nTOzWriter.addDocument(doc);                 // #2
      }
    }

    aTOmWriter.close();
    nTOzWriter.close();

    searchers = new IndexSearcher[2];
    searchers[0] = new IndexSearcher(aTOmDirectory);
    searchers[1] = new IndexSearcher(nTOzDirectory);
  }

  public void testMulti() throws Exception {

    MultiSearcher searcher = new MultiSearcher(searchers);

    TermRangeQuery query = new TermRangeQuery("animal",   // #3
                                              "h",        // #3
                                              "t",        // #3
                                              true, true);// #3

    TopDocs hits = searcher.search(query, 10);
    assertEquals("tarantula not included", 12, hits.totalHits);
  }

  /*
    #1 Create two directories
    #2 Index halves of the alphabet
    #3 Search both indexes
  */
}


```
