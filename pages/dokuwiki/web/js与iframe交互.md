title: js与iframe交互 

#  js与iframe交互 
怎么获取页面中iframe标签中document对象?
document.getElementById('iframeId').contentWindow.document.getElementById(iframe里面的元素)
document.getElementById('iframeId').contentWindow.document.body

IFrame 加载网页完成事件监听?
document.getElementById('iframeId').onload=function(){ alert("加载完成")；}
