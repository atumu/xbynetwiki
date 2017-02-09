modifyAt:2016-12-18 00:52:37
title:Essential SQLAlchemy2th学习笔记之自动生成代码
createAt:2016-12-18 00:52:37
location:python/Essential_SQLAlchemy2th学习笔记之自动生成代码
author:xbynet

```
pip install sqlacodegen
```
sqlacodegen支持从现有数据库自动生成ORM代码，并支持一对多，一对一，多对多的关联关系。

```
#生成整个库的代码
sqlacodegen sqlite:///Chinook_Sqlite.sqlite
#指定表
sqlacodegen sqlite:///Chinook_Sqlite.sqlite --tables Artist,Track
#保存到指定文件
sqlacodegen sqlite:///Chinook_Sqlite.sqlite --tables Artist,Track > db.py
```
