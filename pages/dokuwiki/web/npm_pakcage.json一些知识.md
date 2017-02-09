title: npm_pakcage.json一些知识 

#  npm pakcage.json一些知识 
在package.json中配置scripts选项
```

"scripts": {
    "dev": "webpack-dev-server --devtool eval-source-map --progress --colors --hot --inline --content-base ./dist",
    "build": "webpack --progress --colors"
  },

```
然后通过 **npm run 名字** 来运行对应命令。