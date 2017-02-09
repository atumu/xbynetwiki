title: simplejson 

#  simplejson 
官网：https://simplejson.readthedocs.io/en/latest/
simplejson is a simple, fast, extensible JSON encoder/decoder for Python
当前版本3.9.0

pip install simplejson

```

>>> import simplejson as json
>>> json.dumps(['foo', {'bar': ('baz', None, 1.0, 2)}])
'["foo", {"bar": ["baz", null, 1.0, 2]}]'
>>> print(json.dumps("\"foo\bar"))
"\"foo\bar"
>>> print(json.dumps(u'\u1234'))
"\u1234"
>>> print(json.dumps('\\'))
"\\"
>>> print(json.dumps({"c": 0, "b": 0, "a": 0}, sort_keys=True))
{"a": 0, "b": 0, "c": 0}
>>> from simplejson.compat import StringIO
>>> io = StringIO()
>>> json.dump(['streaming API'], io)
>>> io.getvalue()

```
Compact encoding:
```

>>> import simplejson as json
>>> obj = [1,2,3,{'4': 5, '6': 7}]
>>> json.dumps(obj, separators=(',', ':'), sort_keys=True)
'[1,2,3,{"4":5,"6":7}]'

```

Pretty printing:
```

>>> import simplejson as json
>>> print(json.dumps({'4': 5, '6': 7}, sort_keys=True, indent=4 * ' '))
{
    "4": 5,
    "6": 7
}

```

Decoding JSON:
```

>>> import simplejson as json
>>> obj = [u'foo', {u'bar': [u'baz', None, 1.0, 2]}]
>>> json.loads('["foo", {"bar":["baz", null, 1.0, 2]}]') == obj
True
>>> json.loads('"\\"foo\\bar"') == u'"foo\x08ar'
True
>>> from simplejson.compat import StringIO
>>> io = StringIO('["streaming API"]')
>>> json.load(io)[0] == 'streaming API'
True

```

Specializing JSON object decoding:
```

>>> import simplejson as json
>>> def as_complex(dct):
...     if '__complex__' in dct:
...         return complex(dct['real'], dct['imag'])
...     return dct
...
>>> json.loads('{"__complex__": true, "real": 1, "imag": 2}',
...     object_hook=as_complex)
(1+2j)
>>> import decimal
>>> json.loads('1.1', parse_float=decimal.Decimal) == decimal.Decimal('1.1')
True

```

Specializing JSON object encoding:
```

>>> import simplejson as json
>>> def encode_complex(obj):
...     if isinstance(obj, complex):
...         return [obj.real, obj.imag]
...     raise TypeError(repr(obj) + " is not JSON serializable")
...
>>> json.dumps(2 + 1j, default=encode_complex)
'[2.0, 1.0]'
>>> json.JSONEncoder(default=encode_complex).encode(2 + 1j)
'[2.0, 1.0]'
>>> ''.join(json.JSONEncoder(default=encode_complex).iterencode(2 + 1j))
'[2.0, 1.0]'

```

Using simplejson.tool from the shell to validate and pretty-print:
```

$ echo '{"json":"obj"}' | python -m simplejson.tool
{
    "json": "obj"
}
$ echo '{ 1.2:3.4}' | python -m simplejson.tool

```
Expecting property name enclosed in double quotes: line 1 column 3 (char 2)