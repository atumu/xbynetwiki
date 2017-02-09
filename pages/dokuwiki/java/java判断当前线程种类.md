title: java判断当前线程种类 

#  java判断当前线程种类 
```

    /**判断是否处于Swing EDT线程中
     * @return
     */
    private boolean isEDT(){
    	return Thread.currentThread().getName().startsWith("AWT-EventQueue") ;
    }
    private boolean isMain(){
    	return Thread.currentThread().getName().equals("main") ;
    }

```