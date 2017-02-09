title: flask获取get与post请求_form_json_files_的区别 

#  Flask获取GET与POST请求(form,json,files)的区别 
GET获取参数：
request.args.get('key','default')

POST获取参数分为几种情形：
以表单形式提交的数据request.form[' '],
文件上传：request.files[' ']
以JSON形式提交的数据mimetype为application/json的获取request.json或request.get_json()