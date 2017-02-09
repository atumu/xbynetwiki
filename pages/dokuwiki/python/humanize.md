title: humanize 

#  humanize人性化函数库 
https://github.com/jmoiron/humanize/
pip install humanize

Integer humanization:
```

>>> import humanize
>>> humanize.intcomma(12345)
'12,345'
>>> humanize.intword(123455913)
'123.5 million'
>>> humanize.apnumber(4)
'four'

```
Date & time humanization:
```

>>> import datetime
>>> humanize.naturalday(datetime.datetime.now())
'today'
>>> humanize.naturaldelta(datetime.timedelta(seconds=1001))
'16 minutes'
>>> humanize.naturalday(datetime.datetime.now() - datetime.timedelta(days=1))
'yesterday'

```
Filesize humanization:
```

>>> humanize.naturalsize(1000000)
'1.0 MB'
>>> humanize.naturalsize(1000000, binary=True)
'976.6 KiB'
>>> humanize.naturalsize(1000000, gnu=True)
'976.6K'

```

Localization
How to change locale in runtime
```

>>> print humanize.naturaltime(datetime.timedelta(seconds=3))
3 seconds ago
>>> _t = humanize.i18n.activate('ru_RU')
>>> print humanize.naturaltime(datetime.timedelta(seconds=3))
3 секунды назад
>>> humanize.i18n.deactivate()
>>> print humanize.naturaltime(datetime.timedelta(seconds=3))
3 seconds ago

```
