title: sqlparse 

#  sqlparse 
https://sqlparse.readthedocs.io/en/latest/intro/
A non-validating SQL parser module for Python
$ pip install sqlparse

```

#split
>>> import sqlparse
>>> sql = 'select * from foo; select * from bar;'
>>> sqlparse.split(sql)
[u'select * from foo; ', u'select * from bar;']

#format
>>> sql = 'select * from foo where id in (select id from bar);'
>>> print sqlparse.format(sql, reindent=True, keyword_case='upper')
SELECT *
FROM foo
WHERE id IN
  (SELECT id
   FROM bar);
   
#parse
>>> sql = 'select * from "someschema"."mytable" where id = 1'
>>> parsed = sqlparse.parse(sql)
>>> parsed
(<Statement 'select...' at 0x9ad08ec>,)
>>> stmt = parsed[0]  # grab the Statement object
>>> stmt.tokens


```