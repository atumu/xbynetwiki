title: python学习笔记60 

#  Python开源库之URL操作库 
furl:url操作库.github:https://github.com/gruns/furl。 安装：pip install furl
##  furl 
API:https://github.com/gruns/furl/blob/master/API.md
###  快速入门 
scheme, username, password, and host are strings or None. port is an integer or None.
```

>>> f = furl('http://user:pass@www.google.com:99/')
>>> f.scheme, f.username, f.password, f.host, f.port
('http', 'user', 'pass', 'www.google.com', 99)
furl infers the default port for common schemes.

>>> f = furl('https://secure.google.com/')
>>> f.port
443

>>> f = furl('unknown://www.google.com/')
>>> print f.port
None

```

Query arguments are easy. Really easy.
```

>>> from furl import furl
>>> f = furl('http://www.google.com/?one=1&two=2')
>>> f.args['three'] = '3'
>>> del f.args['one']
>>> f.url
'http://www.google.com/?two=2&three=3'

```
Or use furl's inline modification methods.
```

>>> furl('http://www.google.com/?one=1').add({'two':'2'}).url
'http://www.google.com/?one=1&two=2'

>>> furl('http://www.google.com/?one=1&two=2').set({'three':'3'}).url
'http://www.google.com/?three=3'

>>> furl('http://www.google.com/?one=1&two=2').remove(['one']).url
'http://www.google.com/?two=2'

```
Encoding is handled for you.

```

>>> f = furl('http://www.google.com/')
>>> f.path = 'some encoding here'
>>> f.args['and some encoding'] = 'here, too'
>>> f.url
'http://www.google.com/some%20encoding%20here?and+some+encoding=here,+too'

```
Fragments have a path and a query, too.

```

>>> f = furl('http://www.google.com/')
>>> f.fragment.path.segments = ['two', 'directories']
>>> f.fragment.args = {'one':'argument'}
>>> f.url
'http://www.google.com/#two/directories?one=argument'

```
Or get fancy.

```

>>> f = furl('http://www.google.com/search?q=query#1')
>>> f.copy().remove(path=True).set(host='taco.com')
...  .join('/pumps.html').add(fragment_path='party').url
'http://taco.com/pumps.html#party'

```