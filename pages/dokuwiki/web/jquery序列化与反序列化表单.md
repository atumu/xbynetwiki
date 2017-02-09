title: jquery序列化与反序列化表单 

#  jQuery序列化与反序列化表单 
参考：http://www.css88.com/jqapi-1.9/serializeArray/

##  表单序列化 

jQuery有这两个方法`  .serializeArray() 与.serialize(). `
.serializeArray()返回: Array
描述: 将用作提交的表单元素的值编译成拥有name和value对象组成的数组。例如[ { name: a value: 1 }, { name: b value: 2 },...]
一般用于Ajax POST
```

$('form').submit(function() {
  console.log($(this).serializeArray());
  return false;
});

```
.serialize()返回: String
描述: 将用作提交的表单元素的值编译成字符串。如：single=Single&multiple=Multiple&multiple=Multiple3&check=check2&radio=radio1
一般用于GET

主要介绍.serializeArray()方法。
这个方法很方便，但是在多选的情况先会出现一些问题：
**比如select,checkbox。此时返回的是[{multiple:Multiple},{multiple:Multiple3},{check:check1}},{check:check2}]
而我们期望的是[{multiple:[Multiple,Multiple3]},{check:[check1,check2]}]**
而且我们通过ajax提交的请求数据**根对象期望是json对象而不是直接的json数组。**
```

 <select name="multiple" multiple="multiple">
      <option selected="selected">Multiple</option>
      <option>Multiple2</option>
 
      <option selected="selected">Multiple3</option>
    </select><br/>
    <input type="checkbox" name="check" value="check1" checked="checked" id="ch1"/>
 
    <label for="ch1">check1</label>
    <input type="checkbox" name="check" value="check2" checked="checked" id="ch2"/>

```

针对上面说的情形我们写个工具方法用于处理：
```

   /**
     * 序列化表单元素为JSON对象
     * @param form          Form表单id或表单jquery DOM对象
     * @returns {}
     */
    var serialize = function(form){
        var $form = (typeof(form)=="string" ? $("#"+form) : form);
        var dataArray =  $form.serializeArray();
       var result={};
        $(dataArray).each(function(){
            //如果在结果对象中存在这个值，那么就说明是多选的情形。
            if(result[this.name]){
              	//多选的情形，值是数组，直接push
                result[this.name].push(this.value);
            }else{
              	//获取当前表单控件元素
                var element = $form.find("[name='"+ this.name +"']")[0];
              	//获取当前控件类型
                var type = ( element.type || element.nodeName ).toLowerCase();
                //如果控件类型为多选那么值就是数组形式，否则就是单值形式。
                result[this.name] = (/^(select-multiple|checkbox)$/i).test(type) ? [this.value] : this.value;
            }
        });
        return result;
    };

```
结果：
![](/data/dokuwiki/web/pasted/20151130-151418.png)

##  反序列化 
```

 /**
     * 设置表单值
     * @param form          Form表单id或表单jquery DOM对象
     * @param data          json对象，多选时为数组
     * 代码实现参考此开源项目https://github.com/kflorence/jquery-deserialize/
     */
    var deserialize = function(form,data){
        var rcheck = /^(?:radio|checkbox)$/i,
            rselect = /^(?:option|select-one|select-multiple)$/i,
            rvalue = /^(?:button|color|date|datetime|datetime-local|email|hidden|month|number|password|range|reset|search|submit|tel|text|textarea|time|url|week)$/i;

        var $form = (typeof(form)=="string" ? $("#"+form) : form);

        //得到所有表单元素
        function getElements( elements ) {
          //此处elements为jquery对象。这个map函数使用来便利elements数组的.如存在多个form表单，则便利多个form表单
            return elements.map(function(index, domElemen) {
              	//this代表form表单，this.elements获取表单中的DOM数组. jQuery.makeArray 转换一个类似数组的对象成为真正的JavaScript数组。
                return this.elements ? jQuery.makeArray( this.elements ) : this;
              //过滤不启用的选项
            }).filter( ":input:not(:disabled)" ).get();
        }
        //把表单元素转为json对象
        function elementsToJson( elements ) {
            var current,elementsByName = {};
          //elementsByName对象为{控件名：控件元素或控件元素数组}
            jQuery.each( elements, function( i, element ) {
                current = elementsByName[ element.name ];
                elementsByName[ element.name ] = current === undefined ? element :
                	//如果已经是一个数组了，那么就添加，否则构造一个数组
                    ( jQuery.isArray( current ) ? current.concat( element ) : [ current, element ] );
            });
            return elementsByName;
        }

        var elementsJson = elementsToJson(getElements($form));
	//data是一个对象
        for(var key in data){
            var val = data[key];
            var dataArr = [];//更具数据直接构造一个jQUery序列化后的数组形式。
          //判断值是否为数组
            if( $.isArray(val)){
                for(var i= 0,v;v=val[i++];){
                  	//是数组那么就变成多个对象形式
                    dataArr.push({name:key,value:v});
                }
            } else{
              	//不是数组直接构造
                dataArr.push({name:key,value:val});
            }
		
          	//根据数据构造的这个数组进行操作
            for(var m= 0,vObj;vObj=dataArr[m++];){
                var element;
                //如果表单中无元素则跳过
                if ( !( element = elementsJson[vObj.name] ) ) {
                    continue;
                }
              //判断元素是否为数组,暂时获取第一个元素，后面会有迭代赋值。
                var type = element.length?element[0]:element;
              //元素类型
                type = ( type.type || type.nodeName ).toLowerCase();

                var property = null;
              //是单值类型
                if ( rvalue.test( type ) ) {
                    element.value = (typeof(vObj.value)=="undefined" || vObj.value==null)?"":vObj.value;
                  //checkbox
                } else if ( rcheck.test( type ) ) {
                    property = "checked";
		//select
                } else if ( rselect.test( type ) ) {
                    property = "selected";
                }
                //判断类型是否为多选
                if(property) {
                  	//如果是，则迭代多选的元素赋值
                    for(var n= 0,e;e=element[n++];){
                        if(e.value==vObj.value){
                          //设置选中
                            e[property] = true;
                        }
                    }
                }
            }
        }
    };

```