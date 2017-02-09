title: springmvc之表单标签 

#  springmvc之表单标签 
可供参考资料：
http://www.cnblogs.com/liukemng/p/3754211.html spring系列文章
http://haohaoxuexi.iteye.com/blog/1807330
书籍：spring mvc：a tutorial .PaulDeck
首先，在com.demo.web.models包中添加一个模型TagsModel内容如下：
```

package com.demo.web.models;

import java.util.List;
import java.util.Map;

public class TagsModel{
    
    private String username;
    private String password;
    private boolean testBoolean;
    private String[] selectArray;
    private String[] testArray;
    private Integer radiobuttonId;
    private Integer selectId;
    private List<Integer> selectIds;    
    private Map<Integer,String> testMap;
    private String remark;
    
    public void setUsername(String username){
        this.username=username;
    }
    public void setPassword(String password){
        this.password=password;
    }
    public void setTestBoolean(boolean testBoolean){
        this.testBoolean=testBoolean;
    }
    public void setSelectArray(String[] selectArray){
        this.selectArray=selectArray;
    }
    public void setTestArray(String[] testArray){
        this.testArray=testArray;
    }
    public void setRadiobuttonId(Integer radiobuttonId){
        this.radiobuttonId=radiobuttonId;
    }
    public void setSelectId(Integer selectId){
        this.selectId=selectId;
    }
    public void setSelectIds(List<Integer> selectIds){
        this.selectIds=selectIds;
    }
    public void setTestMap(Map<Integer,String> testMap){
        this.testMap=testMap;
    }
    public void setRemark(String remark){
        this.remark=remark;
    }
    
    public String getUsername(){
        return this.username;
    }
    public String getPassword(){
        return this.password;
    }
    public boolean getTestBoolean(){
        return this.testBoolean;
    }
    public String[] getSelectArray(){
        return this.selectArray;
    }
    public String[] getTestArray(){
        return this.testArray;
    }
    public Integer getRadiobuttonId(){
        return this.radiobuttonId;
    }
    public Integer getSelectId(){
        return this.selectId;
    }
    public List<Integer> getSelectIds(){
        return this.selectIds;
    }
    public Map<Integer,String> getTestMap(){
        return this.testMap;
    }
    public String getRemark(){
        return this.remark;
    }
    
}

```
其次，在包com.demo.web.controllers添加一个TagsController内容如下：
```

@Controller
@RequestMapping(value = "/tags")
public class TagsController {
    
    @RequestMapping(value="/test", method = {RequestMethod.GET})
    public String test(Model model){

        if(!model.containsAttribute("contentModel")){        
            
            TagsModel tagsModel=new TagsModel();
            
            tagsModel.setUsername("aaa");
            tagsModel.setPassword("bbb");
            tagsModel.setTestBoolean(true);
            tagsModel.setSelectArray(new String[] {"arrayItem 路人甲"});
            tagsModel.setTestArray(new String[] {"arrayItem 路人甲","arrayItem 路人乙","arrayItem 路人丙"});
            tagsModel.setRadiobuttonId(1);
            tagsModel.setSelectId(2);
            tagsModel.setSelectIds(Arrays.asList(1,2));
            Map<Integer,String> map=new HashMap<Integer,String>();
            map.put(1, "mapItem 路人甲");
            map.put(2, "mapItem 路人乙");
            map.put(3, "mapItem 路人丙");
            tagsModel.setTestMap(map);
            tagsModel.setRemark("备注...");
            
            model.addAttribute("contentModel", tagsModel);
        }
        return "tagstest";
    }
    
}

```
最后，在views文件夹下添加视图tagstest.jsp内容如下：
```

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <form:form modelAttribute="contentModel" method="post">     
        
        input 标签：<form:input path="username"/><br/>
        password 标签：<form:password path="password"/><br/>
        绑定boolean的checkbox 标签：<br/>
        <form:checkbox path="testBoolean"/><br/>
        绑定Array的checkbox 标签：<br/>
        <form:checkbox path="testArray" value="arrayItem 路人甲"/>arrayItem 路人甲
        <form:checkbox path="testArray" value="arrayItem 路人乙"/>arrayItem 路人乙
        <form:checkbox path="testArray" value="arrayItem 路人丙"/>arrayItem 路人丙
        <form:checkbox path="testArray" value="arrayItem 路人丁"/>arrayItem 路人丁<br/>
        绑定Array的checkboxs 标签：<br/>
        <form:checkboxes path="selectArray" items="${contentModel.testArray}"/><br/>
        绑定Map的checkboxs 标签：<br/>
        <form:checkboxes path="selectIds" items="${contentModel.testMap}"/><br/>
        绑定Integer的radiobutton 标签：<br/>
        <form:radiobutton path="radiobuttonId" value="0"/>0
        <form:radiobutton path="radiobuttonId" value="1"/>1
        <form:radiobutton path="radiobuttonId" value="2"/>2<br/>
        绑定Map的radiobuttons 标签：<br/>
        <form:radiobuttons path="selectId" items="${contentModel.testMap}"/><br/>
        绑定Map的select 标签：<br/>
        <form:select path="selectId" items="${contentModel.testMap}"/><br/>
        不绑定items数据直接在form:option添加的select 标签：<br/>
        <form:select path="selectId">  
           <option>请选择人员</option>
           <form:option value="1">路人甲</form:option>
           <form:option value="2">路人乙</form:option>
           <form:option value="3">路人丙</form:option>
        </form:select><br/>
        不绑定items数据直接在html的option添加的select 标签：<br/>
        <form:select path="selectId">  
           <option>请选择人员</option> 
           <option value="1">路人甲</option>
           <option value="2">路人乙</option>
           <option value="3">路人丙</option>  
        </form:select><br/>
        用form:option绑定items的select 标签：<br/>
        <form:select path="selectId">  
            <option/>请选择人员
            <form:options items="${contentModel.testMap}"/>  
        </form:select><br/>
        textarea 标签：
        <form:textarea path="remark"/><br/>

        <input type="submit" value="Submit" />
        
    </form:form>  
</body>
</html>

```
##  各个标签的使用方法 
1.要使用Spring MVC提供的表单标签，首先需要在视图页面添加：
```

<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

```
2.**form标签**：
```

<form:form modelAttribute="contentModel" method="post">

```
` modelAttribute `属性**指定该form绑定的是哪个Model**，当指定了对应的Model后就可以在form标签内部其它表单标签上通过为path指定Model属性的名称来绑定Model中的数据了
3.input标签：
<form:input path="username"/>
会生成一个type为text的Html input标签，通过` path `属性来**指定要绑定的Model中的属性**。

4.password标签：
<form:password path="password"/>
会生成一个type为password的Html input标签

5.checkbox标签：
会生成一个type为checkbox的Html input标签，**支持绑定boolean、数组、List或Set类型**的数据。
绑定boolean数据会生成一个复选框，当boolean为true该复选框为选定状态，false为不选定状态。
<form:checkbox path="testBoolean"/>
**绑定数组、List或Set类型的数据（以数组作为演示）如果绑定的数据中有对应checkbox指定的value时则为选定状态，反之为不选定状态：**
绑定Array的checkbox 标签：<br/>
<form:checkbox path="testArray" value="arrayItem 路人甲"/>arrayItem 路人甲
<form:checkbox path="testArray" value="arrayItem 路人乙"/>arrayItem 路人乙
<form:checkbox path="testArray" value="arrayItem 路人丙"/>arrayItem 路人丙
<form:checkbox path="testArray" value="arrayItem 路人丁"/>arrayItem 路人丁
![](/data/dokuwiki/spring/pasted/20150812-035221.png)

6.**checkboxs标签**：
会根据绑定的` items `数据生成一组对应的type为checkbox的Html input标签，**绑定的数据可以是数组、集合或Map**，其中checkboxs的path属性也必指定，当path中的数据有和items中的数据值同的时候对应的checkbox为选定状态，反之为不选定状态。
绑定集合数据（以数组作为演示）：
```

绑定Array的checkboxs 标签：<br/>
<form:checkboxes path="selectArray" items="${contentModel.testArray}"/>

```
` 这里需要注意的是当使用EL表达式绑定时需要连Model的名称一起指定如${contentModel.testArray}而不能像path一样只指定Model对应的属性名称。 `
**但通常情况下我们需要的是checkbox显示的是名称，但选择后提交的是对应名称的值，比如id，我们就可以通过绑定Map来实现这个功能：**
```

绑定Map的checkboxs 标签：<br/>
<form:checkboxes path="selectIds" items="${contentModel.testMap}"/>

```
**7.radiobutton标签：**
会生成一个type为radio的Html input标签，如果绑定的数据的值对应radiobutton指定的value时则为选定状态，反之为不选定状态：
绑定Integer的radiobutton 标签：<br/>
<form:radiobutton path="radiobuttonId" value="0"/>0
<form:radiobutton path="radiobuttonId" value="1"/>1
<form:radiobutton path="radiobuttonId" value="2"/>2
**8.radiobuttons标签：**
会根据绑定的` items `数据生成一组对应的type为radio的Html input标签，**绑定的items数据可以是数组、集合或Map**，其中radiobuttons的path属性也必指定，当path的值和items中的某条数据值相同的时候对应的radio为选定状态，反之为不选定状态，用法和checkboxs很相似。但要注意的是：checkboxs的path绑定的是集合radiobuttons的path绑定的是单个值：
` 绑定Map `的radiobuttons 标签：<br/>
```

<form:radiobuttons path="selectId" items="${contentModel.testMap}"/>

```
**9.select标签：**
会生成一个Html select标签，绑定的items数据可以是数组、集合或Map会根据items的内容生成select里面的option选项，当path的值和items中的某条数据值相同的时候对应的option为选定状态，反之为不选定状态，用法与radiobuttons很相似：
```

绑定Map的select 标签：<br/>
<form:select path="selectId" items="${contentModel.testMap}"/>

```
![](/data/dokuwiki/spring/pasted/20150812-035729.png)
上面的是根据指定的items自动生成的option选项，但我们也可以不指定items手动添加select的option选项：
不绑定items数据直接在form:option添加的select 标签：<br/>
```

<form:select path="selectId">  
   <option>请选择人员</option>
   <form:option value="1">路人甲</form:option>
   <form:option value="2">路人乙</form:option>
   <form:option value="3">路人丙</form:option>
</form:select>

```
**10.textarea标签：**
<form:textarea path="remark"/>
11.errors标签：
<form:errors path="要绑定的错误对象属性名" />