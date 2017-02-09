createAt:2017-01-15 13:24:49
author:xbynet
modifyAt:2017-01-15 14:06:51
location:data/Python数据分析之Pandas入门
title:Python数据分析学习笔记之Pandas入门

pandas(Python data analysis)是一个Python数据分析的开源库。
pandas两种数据结构：DataFrame和Series

安装：pandas依赖于NumPy,python-dateutil,pytz
pip install pandas

# DataFrame
DataFrame是一种带标签的二维对象。与excel表格或关系数据库中的表非常神似。可以用以下方式来创建DataFrame：

* 从另一个DataFrame来创建DataFrame
* 从具有二维形状的NumPy数组或者数组的复合结构来生成DataFrame
* 可以用Series来创建DataFrame
* DataFrame可以从类似CSV之类的文件来生成

准备数据资料:http://www.exporedata.net/Downloads/WHO-Data-Set 下载一个csv数据文件。
```
from pandas.io.parsers import read_csv

df = read_csv("WHO_first9cols.csv")
print "Dataframe", df
print "Shape", df.shape
print "Length", len(df)
print "Column Headers", df.columns
print "Data types", df.dtypes
print "Index", df.index
print "Values", df.values

```

注意：DataFrame带有一个索引，类似于关系数据库中的主键。我们既可以手动创建，也可以自动创建。访问df.index
如果需要遍历数据，请使用df.values获取所有值，非数字的数值在被输出时标记为nan。

# Series
Series是一个由不同类型元素组成的一维数组，该数据结构也具有标签。可以通过以下方式创建Series数据结构：

* 由Python字典来创建
* 由NumPy数组来创建
* 由单个标量值来创建

创建Series数据结构时，可以向构造函数递交一组轴标签，这些标签通常称为索引。
对DataFrame列执行查询操作时，会返回一个Series
```
from pandas.io.parsers import read_csv
import numpy as np

df = read_csv("WHO_first9cols.csv")
#这里对DataFrame列进行查询操作，返回一个Series
country_col = df["Country"]
print "Type df", type(df)
print "Type country col", type(country_col)

print "Series shape", country_col.shape
print "Series index", country_col.index
print "Series values", country_col.values
print "Series name", country_col.name

print "Last 2 countries", country_col[-2:]
print "Last 2 countries type", type(country_col[-2:])
#NumPy的函数同样适用于pandas的DataFrame和Series
print "df signs", np.sign(df)
last_col = df.columns[-1]
print "Last df column signs", last_col, np.sign(df[last_col])

print np.sum(df[last_col] - df[last_col].values)
```

# 利用pandas查询数据
数据准备：pip install Quandl 或者手动从http://www.quandl.com/SIDC/SUNSPOTS_A-Sunspot-Numbers-Annual 下载csv文件。
```
import Quandl

# Data from http://www.quandl.com/SIDC/SUNSPOTS_A-Sunspot-Numbers-Annual
# PyPi url https://pypi.python.org/pypi/Quandl
sunspots = Quandl.get("SIDC/SUNSPOTS_A")
print "Head 2", sunspots.head(2) 
print "Tail 2", sunspots.tail(2)

last_date = sunspots.index[-1]
print "Last value", sunspots.loc[last_date]

print "Values slice by date", sunspots["20020101": "20131231"]

print "Slice from a list of indices", sunspots.iloc[[2, 4, -4, -2]]

print "Scalar with Iloc", sunspots.iloc[0, 0]
print "Scalar with iat", sunspots.iat[1, 0]

print "Boolean selection", sunspots[sunspots > sunspots.mean()]
print "Boolean selection with column label", sunspots[sunspots.Number > sunspots.Number.mean()]

```

DataFrame的统计函数
describe、count、mad、median、min、max、,pde、std、var、skew、kurt

# DataFrame分组与聚合
```
import pandas as pd
from numpy.random import seed
from numpy.random import rand
from numpy.random import random_integers
import numpy as np

seed(42)

df = pd.DataFrame({'Weather' : ['cold', 'hot', 'cold', 'hot',
   'cold', 'hot', 'cold'],
   'Food' : ['soup', 'soup', 'icecream', 'chocolate',
   'icecream', 'icecream', 'soup'],
   'Price' : 10 * rand(7), 'Number' : random_integers(1, 9, size=(7,))})

print df
weather_group = df.groupby('Weather')

i = 0

for name, group in weather_group:
   i = i + 1
   print "Group", i, name
   print group

print "Weather group first\n", weather_group.first()
print "Weather group last\n", weather_group.last()
print "Weather group mean\n", weather_group.mean()

wf_group = df.groupby(['Weather', 'Food'])
print "WF Groups", wf_group.groups
#通过agg方法，可以对数据组施加一系列的NumPy函数。
print "WF Aggregated\n", wf_group.agg([np.mean, np.median])
```

# DataFrame的串联与附加操作
数据库的数据表有内部连接和外部连接。DataFrame也有类似操作，即串联和附加。
**函数concat()的作用是串联DataFrame，追加数据行使用append()函数。**
例如
```
pd.concat([df[:3],df[3:]])
df[:3].append(df[5:])
```

pandas提供merge()或DataFrane的join()方法都能实现类似数据库的连接操作功能。默认情况下join()方法会按照索引进行连接，不过，有时候这不符合我们的要求。
数据准备：
tips.csv
```
EmpNr,Amount
5,10
9,5
7,2.5
```
dest.csv
```
EmpNr,Dest
5,The Hague
3,Amsterdam
9,Rotterdam

```

```
dests = pd.read_csv('dest.csv')
tips = pd.read_csv('tips.csv')
#使用merge()函数按照员工编号进行连接处理
print "Merge() on key\n", pd.merge(dests, tips, on='EmpNr')
#用join()方法执行连接操作时，需要使用后缀来指示左、右操作对象。
print "Dests join() tips\n", dests.join(tips, lsuffix='Dest', rsuffix='Tips')
#用merge()执行内部连接时，更显示的方法如下
print "Inner join with merge()\n", pd.merge(dests, tips, how='inner')
#稍作修改便变成完全外部连接，缺失的数据变为NaN
print "Outer join\n", pd.merge(dests, tips, how='outer')

```

# 处理缺失的数据
缺失的数据变为NaN(非数字)，还有一个类似的符号NaT(非日期). 可以使用pandas的两个函数来进行判断isnull(),notnull(), fillna()方法可以用一个标量值来替换缺失的数据。
```
import pandas as pd
import numpy as np

df = pd.read_csv('WHO_first9cols.csv')
# Select first 3 rows of country and Net primary school enrolment ratio male (%)
df = df[['Country', df.columns[-2]]][:2]
print "New df\n", df
print "Null Values\n", pd.isnull(df)
print "Total Null Values\n", pd.isnull(df).sum()
print "Not Null Values\n", df.notnull()
print "Last Column Doubled\n", 2 * df[df.columns[-1]]
print "Last Column plus NaN\n", df[df.columns[-1]] + np.nan
print "Zero filled\n", df.fillna(0)
```

# 处理日期数据
http://pandas.pydata.org/pandas-docs/stable/timeseries.html
各种频率(freq)短码对照表:

* B	business day frequency
* C	custom business day frequency (experimental)
* D	calendar day frequency
* W	weekly frequency
* M	month end frequency
* SM	semi-month end frequency (15th and end of month)
* BM	business month end frequency
* CBM	custom business month end frequency
* MS	month start frequency
* SMS	semi-month start frequency (1st and 15th)
* BMS	business month start frequency
* CBMS	custom business month start frequency
* Q	quarter end frequency
* BQ	business quarter endfrequency
* QS	quarter start frequency
* BQS	business quarter start frequency
* A	year end frequency
* BA	business year end frequency
* AS	year start frequency
* BAS	business year start frequency
* BH	business hour frequency
* H	hourly frequency
* T, min	minutely frequency
* S	secondly frequency
* L, ms	milliseconds
* U, us	microseconds
* N	nanoseconds

```
import pandas as pd
from pandas.tseries.offsets import DateOffset
import sys

print "Date range", pd.date_range('1/1/1900', periods=42, freq='D')

try:
   print "Date range", pd.date_range('1/1/1677', periods=4, freq='D')
except:
   etype, value, _ = sys.exc_info()
   print "Error encountered", etype, value

offset = DateOffset(seconds=2 ** 63/10 ** 9)
mid = pd.to_datetime('1/1/1970')
print "Start valid range", mid - offset
print "End valid range", mid + offset
print pd.to_datetime(['1900/1/1', '1901.12.11'])

print "With format", pd.to_datetime(['19021112', '19031230'], format='%Y%m%d')

print "Illegal date", pd.to_datetime(['1902-11-12', 'not a date'])
print "Illegal date coerced", pd.to_datetime(['1902-11-12', 'not a date'], coerce=True)
```

# 据透视表(pivot_table)
数据透视表可以用来汇总数据。pivot_table()函数及相应的DataFrame方法。
```
import pandas as pd
from numpy.random import seed
from numpy.random import rand
from numpy.random import random_integers
import numpy as np

seed(42)
N = 7
df = pd.DataFrame({
   'Weather' : ['cold', 'hot', 'cold', 'hot',
   'cold', 'hot', 'cold'],
   'Food' : ['soup', 'soup', 'icecream', 'chocolate',
   'icecream', 'icecream', 'soup'],
   'Price' : 10 * rand(N), 'Number' : random_integers(1, 9, size=(N,))})

print "DataFrame\n", df
#cols指定需要聚合的列，aggfunc指定聚合函数。
print pd.pivot_table(df, cols=['Food'], aggfunc=np.sum)
```