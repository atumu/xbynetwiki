title: jquery_validate 

#  jQuery validate 
参考：http://www.runoob.com/jquery/jquery-plugin-validate.html
依赖：
JQuery,"JQuery.validate","JQuery.validate.extra","JQuery.validate.message"
##  Jquery Validate 验证规则 
(1)required:true              必输字段
(2)remote:”check.php”         使用ajax方法调用check.php验证输入值
(3)email:true                 必须输入正确格式的电子邮件
(4)url:true                   必须输入正确格式的网址
(5)date:true                  必须输入正确格式的日期
(6)dateISO:true               必须输入正确格式的日期(ISO)，例如：2009-06-23，1998/01/22 只验证格式，不验证有效性
(7)number:true                必须输入合法的数字(负数，小数)
(8)digits:true                必须输入整数
(9)creditcard:                必须输入合法的信用卡号
(10)equalTo:”#field”          输入值必须和#field相同
(11)accept:                   输入拥有合法后缀名的字符串（上传文件的后缀）
(12)maxlength:5               输入长度最多是5的字符串(汉字算一个字符)
(13)minlength:10              输入长度最小是10的字符串(汉字算一个字符)
(14)rangelength:[5,10]        输入长度必须介于 5 和 10 之间的字符串”)(汉字算一个字符)
(15)range:[5,10]              输入值必须介于 5 和 10 之间
(16)max:5                     输入值不能大于5
(17)min:10                    输入值不能小于10
##  Jquery Validate 自定义验证规则 
addMethod(name,method,message)方法：
参数name 是添加的方法的名字
参数method是一个函数,接收三个参数(value,element,param) value 是元素的值,element是元素本身param
是参数,我们可以用addMethod 来添加除built-in Validation methods 之外的验证方法比如有一个字段,只
能输一个字母,范围是a-f,写法如下:
```

//用于select控件校验
	$.validator.addMethod("selectValid",function(value,element,params){
		if(value=="0"){
			return false;
		}
		return true;
		},"请选择类别");

```
用的时候,比如有个表单字段的id=”username”,则在rules 中写
```

$("#menuForm").validate({
				rules:{
					role:{
						selectValid:"a"
					}
				},
				messages:{
					role:{
						selectValid:"请选择角色类别"
					}
				}
			});

```
方法
addMethod 的第一个参数,就是添加的验证方法的名子,这时是af
addMethod 的第三个参数,就是自定义的错误提示,这里的提示为:”必须是一个字母,且a-f”
addMethod 的第二个参数,是一个函数,这个比较重要,决定了用这个验证方法时的写法
如果只有一个参数,直接写,如果selectValid:”a”,那么a 就是这个唯一的参数,如果多个参数,用在[]里,用逗号分开
##  Jquery Validate submit 提交 
submitHandler:
通过验证后运行的函数,里面要加上表单提交的函
数,否则表单不会提交
```

$(".selector").validate({
   submitHandler:function(form) {
$(form).ajaxSubmit();          //用Jquery Form的函数
   }
})

```
##  Jquery Validate error 错误提示dom 
.errorPlacement：Callback Default: 把错误信息放在验证的元素后面
指明错误放置的位置，默认情况是：error.appendTo(element.parent());即把错误信息放在验证的元素后面
errorPlacement: function(error, element) {
error.appendTo(element.parent());
}
设置错误提示的样式，可以增加图标显示，like：
```

input.error { border: 1px solid red; }
label.error {
background:url(“./demo/images/unchecked.gif”) no-repeat 0px0px;
  padding-left: 16px;
  padding-bottom: 2px;
  font-weight: bold;
  color: #EA5200;
}

```

##  使用： 
添加css:
```

.error{
	 font-size:14px; 
	 font-weight:bold;
	 color:red;
}

```
```

   //站点表单校验
    function validate_site(fromName){
    	$("#"+fromName).validate({
			rules : {
				siteName : "required",
				sitePath : {
        			required:true,
        			validate_path:""
        		},
        		siteLogo : {
        			validate_path:""
        		},
        		siteAddTemplateName : "required",
        		channelAddTemplateName : "required",
        		articleAddTemplateName : "required",
        		siteTemplateName : "required",
        		channelTemplateName : "required",
        		articleTemplateName : "required",
			},
			messages : {
				siteName : "请填写站点名称",
				siteAddTemplateName : "请选择站点模板",
				channelAddTemplateName : "请选择站点下栏目默认模板",
				articleAddTemplateName : "请选择站点栏目下文章默认模板",
				siteTemplateName : "请选择站点模板",
	        	channelTemplateName : "请选择站点下栏目默认模板",
	        	articleTemplateName : "请选择站点栏目下文章默认模板"
			}
		})
    }
    function CreateOrEditResources(flag,resource_id){
		 //初始化校验
	           initValidate();
	            //监听【保存】按钮事件
	            $("#btn_resourcesEdit_save").on("click", function() {
	            		//validate_site("form_testwwwAdd");
	            	if ($("#form_resourcesEdit").valid()) {//校验站点

```
                          
##  将校验规则写到控件中 
 Validate forms like you've never validated before!
参考http://jqueryvalidation.org/documentation/
```

<form class="cmxform" id="commentForm" method="get" action="">
  <fieldset>
    <legend>Please provide your name, email address (won't be published) and a comment</legend>
    <p>
      <label for="cname">Name (required, at least 2 characters)</label>
      <input id="cname" name="name" minlength="2" type="text" required>
    </p>
    <p>
      <label for="cemail">E-Mail (required)</label>
      <input id="cemail" type="email" name="email" required>
    </p>
    <p>
      <label for="curl">URL (optional)</label>
      <input id="curl" type="url" name="url">
    </p>
    <p>
      <label for="ccomment">Your comment (required)</label>
      <textarea id="ccomment" name="comment" required></textarea>
    </p>
    <p>
      <input class="submit" type="submit" value="Submit">
    </p>
  </fieldset>
</form>
<script>
$("#commentForm").validate();
</script>

```
##  远程验证 
remote：URL
使用 ajax 方式进行验证，默认会提交当前验证的值到远程地址，如果需要提交其他的值，可以使用 data 选项。
```

remote: "check-email.php"
remote: {
    url: "check-email.php",     //后台处理程序
    type: "post",               //数据发送方式
    dataType: "json",           //接受数据格式   
    async:false,   //改为同步，异步会导致很多业务相关问题。
    data: {                     //要传递的数据
        username: function() {
            return $("#username").val();
        }
    }
}

```
` **远程地址只能输出 "true" 或 "false"，不能有其他输出。** `

```

$("#loginform").validate({
				rules:{
					captchaCode:{
		               required:true,
		               remote:{ 
		                  type:"POST",  
		                  url: "captcha/valid", 
                                   async:false,   //改为同步，异步会导致很多业务相关问题。
		                  data:{  
		                	  captchaCode:function(){return $("#captchaCode").val();}  
		                  }  
		               }
		       	    },
		       	 name:{
			       		required:true
			       	},
			       	password:{
			       		required:true
			       	}
				},
			 	messages: {
			 		captchaCode:{
			     		remote:"验证码错误"
			     	}
			 	}
			});
		$("#loginform").submit(function() {
			var isValid = $("#loginform").valid();

			return isValid;
		}); 

```

##  自定义验证规则 
```

	//用于select控件校验
	$.validator.addMethod("selectValid",function(value,element,params){
		if(value&&value!=""&&value!="0"){
			return true;
		}
		return false;
		},"请选择类别");
	//用于校验不能含有,号的字符串
	$.validator.addMethod("noCommaValid",function(value,element,params){
		if(value&&value!=""&&value.indexOf(",")<=0){
			return true;
		}
		return false;
		},"不能含有逗号");

```
```

		$("#menuForm").validate({
				rules:{
					role:{
						selectValid:"a" //初始化验证
					},
					isShow:{
						selectValid:"a"
					}
				},
				messages:{
					role:{
						selectValid:"请选择角色类别"
					}
				}
			});

```
##  附录：常用的自定义验证规则 

```

// 字符验证
jQuery.validator.addMethod(“stringCheck”, function(value, element) {
return this.optional(element) || /^[u0391-uFFE5w]+$/.test(value);
}, ”只能包括中文字、英文字母、数字和下划线”);
// 中文字两个字节
jQuery.validator.addMethod(“byteRangeLength”, function(value, element, param) {
var length = value.length;
for(var i = 0; i < value.length; i++){
if(value.charCodeAt(i) > 127){
length++;
}
}
return this.optional(element) || ( length >= param[0] && length <= param[1] );
}, ”请确保输入的值在3-15个字节之间(一个中文字算2个字节)”);
// 身份证号码验证
jQuery.validator.addMethod(“isIdCardNo”, function(value, element) {
return this.optional(element) || isIdCardNo(value);
}, ”请正确输入您的身份证号码”);
// 手机号码验证
jQuery.validator.addMethod(“isMobile”, function(value, element) {
var length = value.length;
var mobile = /^(((13[0-9]{1})|(15[0-9]{1}))+d{8})$/;
return this.optional(element) || (length == 11 && mobile.test(value));
}, ”请正确填写您的手机号码”);
// 电话号码验证
jQuery.validator.addMethod(“isTel”, function(value, element) {
var tel = /^d{3,4}-?d{7,9}$/;    //电话号码格式010-12345678
return this.optional(element) || (tel.test(value));
}, ”请正确填写您的电话号码”);
// 联系电话(手机/电话皆可)验证
jQuery.validator.addMethod(“isPhone”, function(value,element) {
var length = value.length;
var mobile = /^(((13[0-9]{1})|(15[0-9]{1}))+d{8})$/;
var tel = /^d{3,4}-?d{7,9}$/;
return this.optional(element) || (tel.test(value) || mobile.test(value));
}, ”请正确填写您的联系电话”);
// 邮政编码验证
jQuery.validator.addMethod(“isZipCode”, function(value, element) {
var tel = /^[0-9]{6}$/;
return this.optional(element) || (tel.test(value));
}, ”请正确填写您的邮政编码”);

function isIdCardNo(num) {
var factorArr = newArray(7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2,1);
var parityBit=newArray(“1″,”0″,”X”,”9″,”8″,”7″,”6″,”5″,”4″,”3″,”2″);
var varArray = new Array();
var intValue;
var lngProduct = 0;
var intCheckDigit;
var intStrLen = num.length;
var idNumber = num;
// initialize
if ((intStrLen != 15) && (intStrLen!= 18)) {
return false;
}
// check and set value
for(i=0;i<intStrLen;i++) {
varArray[i] = idNumber.charAt(i);
if ((varArray[i] < ’0′ || varArray[i]> ’9′) && (i != 17)){
return false;
} else if (i < 17) {
varArray[i] = varArray[i] * factorArr[i];
}
}
if (intStrLen == 18) {
//check date
var date8 = idNumber.substring(6,14);
if (isDate8(date8) == false) {
return false;
}
// calculate the sum of the products
for(i=0;i<17;i++) {
lngProduct = lngProduct + varArray[i];
}
// calculate the check digit
intCheckDigit = parityBit[lngProduct % 11];
// check last digit
if (varArray[17] != intCheckDigit) {
return false;
}
}
else{       //length is 15
//check date
var date6 = idNumber.substring(6,12);
if (isDate6(date6) == false) {
return false;
}
}
return true;
}

function isDate6(sDate) {
if(!/^[0-9]{6}$/.test(sDate)) {
return false;
}
var year, month, day;
year = sDate.substring(0, 4);
month = sDate.substring(4, 6);
if (year < 1700 || year > 2500)return false
if (month < 1 || month > 12) returnfalse
return true
}

function isDate8(sDate) {
if(!/^[0-9]{8}$/.test(sDate)) {
return false;
}
var year, month, day;
year = sDate.substring(0, 4);
month = sDate.substring(4, 6);
day = sDate.substring(6, 8);
var iaMonthDays = [31,28,31,30,31,30,31,31,30,31,30,31]
if (year < 1700 || year > 2500)return false
if (((year % 4 == 0) && (year % 100!= 0)) || (year % 400 == 0)) iaMonthDays[1]=29;
if (month < 1 || month > 12) returnfalse
if (day < 1 || day >iaMonthDays[month - 1]) return false
return true
}
//身份证号码验证   
jQuery.validator.addMethod(“idcardno”, function(value, element){
return this.optional(element) || isIdCardNo(value);
}, “请正确输入身份证号码”);
//字母数字
jQuery.validator.addMethod(“alnum”, function(value, element){
return this.optional(element) ||/^[a-zA-Z0-9]+$/.test(value);
}, “只能包括英文字母和数字”);
 // 邮政编码验证
jQuery.validator.addMethod(“zipcode”, function(value, element){
var tel = /^[0-9]{6}$/;
return this.optional(element) || (tel.test(value));
}, “请正确填写邮政编码”);
  // 汉字
jQuery.validator.addMethod(“chcharacter”, function(value, element){
var tel = /^[u4e00-u9fa5]+$/;
return this.optional(element) || (tel.test(value));
}, “请输入汉字”);
// 字符最小长度验证（一个中文字符长度为2）
jQuery.validator.addMethod(“stringMinLength”, function(value,element, param) {
var length = value.length;
for ( var i = 0; i < value.length; i++) {
if (value.charCodeAt(i) > 127) {
length++;
}
}
return this.optional(element) || (length >=param);
}, $.validator.format(“长度不能小于{0}!”));
// 字符最大长度验证（一个中文字符长度为2）
jQuery.validator.addMethod(“stringMaxLength”, function(value,element, param) {
var length = value.length;
for ( var i = 0; i < value.length; i++) {
if (value.charCodeAt(i) > 127) {
length++;
}
}
return this.optional(element) || (length <=param);
}, $.validator.format(“长度不能大于{0}!”));
// 字符验证
jQuery.validator.addMethod(“string”, function(value, element){
return this.optional(element) ||/^[u0391-uFFE5w]+$/.test(value);
}, “不允许包含特殊符号!”);
// 手机号码验证
jQuery.validator.addMethod(“mobile”, function(value, element){
var length = value.length;
return this.optional(element) || (length == 11&&/^(((13[0-9]{1})|(15[0-9]{1}))+d{8})$/.test(value));
}, “手机号码格式错误!”);
// 电话号码验证
jQuery.validator.addMethod(“phone”, function(value, element){
var tel = /^(d{3,4}-?)?d{7,9}$/g;
return this.optional(element) || (tel.test(value));
}, “电话号码格式错误!”);
// 邮政编码验证
jQuery.validator.addMethod(“zipCode”, function(value, element){
var tel = /^[0-9]{6}$/;
return this.optional(element) || (tel.test(value));
}, “邮政编码格式错误!”);
// 必须以特定字符串开头验证
jQuery.validator.addMethod(“begin”, function(value, element, param){
var begin = new RegExp(“^” + param);
return this.optional(element) || (begin.test(value));
}, $.validator.format(“必须以 {0} 开头!”));
// 验证两次输入值是否不相同
jQuery.validator.addMethod(“notEqualTo”, function(value, element,param) {
return value != $(param).val();
}, $.validator.format(“两次输入不能相同!”));
//验证值不允许与特定值等于
jQuery.validator.addMethod(“notEqual”, function(value, element,param) {
return value != param;
}, $.validator.format(“输入值不允许为{0}!”));
// 验证值必须大于特定值(不能等于)
jQuery.validator.addMethod(“gt”, function(value, element, param){
return value > param;
}, $.validator.format(“输入值必须大于{0}!”));

```

参考:http://www.runoob.com/jquery/jquery-plugin-validate.html
http://blog.csdn.net/sunhuwh/article/details/24089933
http://www.runoob.com/jquery/jquery-plugin-validate.html