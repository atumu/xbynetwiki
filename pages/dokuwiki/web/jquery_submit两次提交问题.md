title: jquery_submit两次提交问题 

#  jquery submit两次提交问题 
在使用如下代码进行表单提交时会出现重复两次提交的情形：
原因是我使用JQueryValidate远程异步验证导致的问题。解决方案：改为远程同步认证即可解决该问题
```

$("#loginform").validate({
				rules : {
					captchaCode : {
						required : true,
						remote : {
							type : "POST",
							url : "captcha/valid",
							async:false,   //改为同步即可解决该问题
							data : {
								captchaCode : function() {
									return $("#captchaCode").val();
								}
							}
						}
					},
					name : {
						required : true
					},
					password : {
						required : true
					}
				},
				messages : {
					captchaCode : {
						remote : "验证码错误"
					}
				}
			});

```
