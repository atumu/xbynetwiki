title: css边框 

#  css边框 
##  多重边框 
利用重复指定` box-shadow `来达到多个边框的效果
在线演示    
```

/*CSS Border with Box-Shadow Example*/
div {
    box-shadow: 0 0 0 6px rgba(0, 0, 0, 0.2), 0 0 0 12px rgba(0, 0, 0, 0.2), 0 0 0 18px rgba(0, 0, 0, 0.2), 0 0 0 24px rgba(0, 0, 0, 0.2);
    height: 200px;
    margin: 50px auto;
    width: 400px
}

```