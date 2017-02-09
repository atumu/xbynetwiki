title: luceneinaction_第2版_学习笔记10 

#  LuceneInAction2rd学习笔记之多线程并行处理 
Lucene已经能够很好的利用多线程机制，**Lucene是线程安全的：它能够完美支持多线程共享IndexSearcher、IndexReader、IndexWriter等对象。同时,Lucene还是线程友好的**：它将同步代码块压缩至最小。
下面将展示如何在索引和搜索期间平衡线程数量
**使用多线程进行索引操作：**
考虑几点：线程数量一般是CPU核心数+1，队列长度一般是4*线程数。
![](/data/dokuwiki/lucene/pasted/20160302-224645.png)
```

/** Drop-in replacement for IndexWriter that uses multiple
 *  threads, under the hood, to index added documents. */

public class ThreadedIndexWriter extends IndexWriter {

  private ExecutorService threadPool;
  private Analyzer defaultAnalyzer;

  private class Job implements Runnable {                       //A
    Document doc;
    Analyzer analyzer;
    Term delTerm;
    public Job(Document doc, Term delTerm, Analyzer analyzer) {
      this.doc = doc;
      this.analyzer = analyzer;
      this.delTerm = delTerm;
    }
    public void run() {                                         //B
      try {
        if (delTerm != null) {
          ThreadedIndexWriter.super.updateDocument(delTerm, doc, analyzer);
        } else {
          ThreadedIndexWriter.super.addDocument(doc, analyzer);
        }
      } catch (IOException ioe) {
        throw new RuntimeException(ioe);
      }
    }
  }

  public ThreadedIndexWriter(Directory dir, Analyzer a,
                             boolean create, int numThreads,
                             int maxQueueSize, IndexWriter.MaxFieldLength mfl)
       throws CorruptIndexException, IOException {
    super(dir, a, create, mfl);
    defaultAnalyzer = a;
    threadPool = new ThreadPoolExecutor(                        //C
          numThreads, numThreads,
          0, TimeUnit.SECONDS,
          new ArrayBlockingQueue<Runnable>(maxQueueSize, false),
          new ThreadPoolExecutor.CallerRunsPolicy());
  }

  public void addDocument(Document doc) {                       //D
    threadPool.execute(new Job(doc, null, defaultAnalyzer));    //D
  }                                                             //D
                                                                //D
  public void addDocument(Document doc, Analyzer a) {           //D
    threadPool.execute(new Job(doc, null, a));                  //D
  }                                                             //D
                                                                //D
  public void updateDocument(Term term, Document doc) {         //D
    threadPool.execute(new Job(doc, term, defaultAnalyzer));    //D
  }                                                             //D
                                                                //D 
  public void updateDocument(Term term, Document doc, Analyzer a) { //D
    threadPool.execute(new Job(doc, term, a));                  //D
  }                                                             //D

  public void close()
      throws CorruptIndexException, IOException {
    finish();
    super.close();
  }

  public void close(boolean doWait)
      throws CorruptIndexException, IOException {
    finish();
    super.close(doWait);
  }

  public void rollback()
      throws CorruptIndexException, IOException {
    finish();
    super.rollback();
  }

  private void finish() {                                       //E
    threadPool.shutdown();
    while(true) {
      try {
        if (threadPool.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS)) {
          break;
        }
      } catch (InterruptedException ie) {
        Thread.currentThread().interrupt();
        throw new RuntimeException(ie);
      }
    }
  }
}

/*
#A Holds one document to be added
#B Does real work to add or update document
#C Create thread pool
#D Have thread pool execute job
#E Shuts down thread pool
*/


```
使用多线程方式安全重启IndexSearcher
```

public class SearcherManager {

  private IndexSearcher currentSearcher;                         //A
  private IndexWriter writer;

  public SearcherManager(Directory dir) throws IOException {     //1
    currentSearcher = new IndexSearcher(IndexReader.open(dir));  //B
    warm(currentSearcher);
  }

  public SearcherManager(IndexWriter writer) throws IOException { //2
    this.writer = writer;
    currentSearcher = new IndexSearcher(writer.getReader());      //C
    warm(currentSearcher);

    writer.setMergedSegmentWarmer(                                   // 3
        new IndexWriter.IndexReaderWarmer() {                        // 3
          public void warm(IndexReader reader) throws IOException {  // 3
            SearcherManager.this.warm(new IndexSearcher(reader));    // 3
          }                                                          // 3
        });                                                          // 3
  }

  public void warm(IndexSearcher searcher)    // D
    throws IOException                        // D
  {}                                          // D

  private boolean reopening;

  private synchronized void startReopen()
    throws InterruptedException {
    while (reopening) {
      wait();
    }
    reopening = true;
  }

  private synchronized void doneReopen() {
    reopening = false;
    notifyAll();
  }

  public void maybeReopen()                      //E
    throws InterruptedException,                 //E
           IOException {                         //E

    startReopen();

    try {
      final IndexSearcher searcher = get();
      try {
        IndexReader newReader = currentSearcher.getIndexReader().reopen();
        if (newReader != currentSearcher.getIndexReader()) {
          IndexSearcher newSearcher = new IndexSearcher(newReader);
          if (writer == null) {
            warm(newSearcher);
          }
          swapSearcher(newSearcher);
        }
      } finally {
        release(searcher);
      }
    } finally {
      doneReopen();
    }
  }

  public synchronized IndexSearcher get() {                      //F
    currentSearcher.getIndexReader().incRef();
    return currentSearcher;
  }    

  public synchronized void release(IndexSearcher searcher)       //G
    throws IOException {
    searcher.getIndexReader().decRef();
  }

  private synchronized void swapSearcher(IndexSearcher newSearcher)
    throws IOException {
    release(currentSearcher);
    currentSearcher = newSearcher;
  }

  public void close() throws IOException {
    swapSearcher(null);
  }
}

/*
#A Current IndexSearcher
#B Create searcher from Directory
#C Create searcher from near-real-time reader
#D Implement in subclass
#E Reopen searcher
#F Returns current searcher
#G Release searcher
*/


```