title: js_checkbox赋值与取值 

#  js checkbox赋值与取值 
赋值：
```

if(data.content.isSummary=='yes'){
					$("#isSummary[value='yes']").prop("checked",true);
				}else{
					$("#isSummary[value='no']").prop("checked",true);
				}

```
取值:
```

$("input[name='isSummary']:checked").val()

```