title: springmvc防止表单重复提交思路 

#  springmvc防止表单重复提交思路 
考虑到现在都是通过ajax提交请求，所以可以采取如下形式：
  * js端：**ajax提交，页面按钮禁用，返回结果后，按钮启用**
  * 服务端：在Session中保存一个表单的唯一编号，将该编号放在一个隐藏域中，同其他数据一同提交。在提交表单后，通过拦截器或其他机制检查唯一编号，如果存在则说明表单是第一次提交，如果不存在则被重复提交（**理由很简单，在第一次提交检查后就会从Session中移除该编号**）。保存编号可以用一个 HashMap。

非Ajax方式：请参考：[[spring:springmvc_flash_attribute_的讲解与使用]], [[spring:spring_mvc_controller间跳转_重定向_传参]]
还可以参考：
http://blog.csdn.net/mylovepan/article/details/38894941
http://www.oschina.net/code/snippet_100825_21906
http://www.oschina.net/question/557540_91355