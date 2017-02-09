title: flask大型项目目录结构 

#  Flask大型项目目录结构 
```

|-flasky
    |-app/#Flask 程序一般都保存在名为 app 的包中;
        |-templates/
        |-static/
        |-main/
            |-\__init\__.py#创建蓝本实例
            |-errors.py#错误处理程序
            |-forms.py
            |-views.py#视图函数
        |-__init__.py#工厂函数，用于生成app实例
        |-email.py#电子邮件支持函数
        |-models.py#数据库模型
    |-migrations/#migrations 文件夹包含数据库迁移脚本
    |-tests/#单元测试
        |-\__init\__.py
        |-test*.py
    |-venv/#venv 文件夹包含 Python 虚拟环境
    |-requirements.txt#列出了所有依赖包,便于在其他电脑中重新生成相同的虚拟环境pip freeze >requirements.txt
    |-config.py#存储配置
    |-manage.py#用于启动程序以及其他的程序任务

```