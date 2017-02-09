title: python学习笔记7 

#  Python学习笔记之编写循环技巧与函数式编程 
  * 主要使用函数:range,zip,enumerate,map,filter,reduce
  * lambda表达式:lambda 参数列表:求值表达式
  * 列表解析表达式：如[c*i for (i,c) in enumerate(S)]
  * 迭代器:next()内置函数,__next__属性,` iter() `内置函数。字典的keys(),items(),values()
  * 生成器与yield
##  主要使用函数 
**range 可以用于循环计数器，返回一个迭代器**
```

for i in range(3): print(i,end=',') #输出0,1,2,不包括3
list(range[-3,3])#得到[-3,-2,-1,0,1,2] 不包括3
for i in range(len('aaaa')):print('aaaa'[i],end=', ')
for i in range(0,10,2): print(i) #输出0,2,4,6,8,

```
  
**zip:主要用于对多个列表进行并行遍历.并将每个对应位置处的元素组合成一个元组，最后返回这些元组组合而成的迭代器**
```

L1=[1,2,3,4]
L2=[5,6,7,8]
zip(L1,L2)#返回一个可迭代对象。
list(zip(L1,L2))#得到[(1, 5), (2, 6), (3, 7), (4, 8)]
dict(zip(L1,L2))#使用zip构造字典,{1: 5, 2: 6, 3: 7, 4: 8}

```


**enumerate:产生偏移和元素**
```

S='spam'
for(offset,item) in enumerate(S):
	print(item,'appears at offset',offset)
输出:
s appears at offset 0
p appears at offset 1
a appears at offset 2
m appears at offset 3

[c*i for (i,c) in enumerate(S)]
输出:
[''.'p','ss','mmm']

```

**map：在序列中映射函数(即对每一个元素应用函数),返回一个迭代器**
```

def inc(x):return x+10
print(list(map(inc,[1,2,3,4])))
输出
[11, 12, 13, 14]

list(map((lambda x:x+10),[1,2,3,4]))

```

**filter:对列表的每一项应用一个返回True/False的函数进行列表过滤,返回一个迭代器**
```

list(filter((lambda x:x>0),range(-5,5))) #输出[1,2,3,4]

```

**reduce：以前面函数的返回值与目前元素为参数传给一个函数，依次进行。最后返回一个值。**
` 注意:reduce已经被移到functools下面了。from functools import reduce `
```

reduce((lambda x,y:x+y),[1,2,3,4])#得到10

```
##  lambda表达式与匿名函数 
lambda 参数列表:表达式
返回一个函数，然后调用这个函数得到表达式的值
注意:lambda中不能用print等。只能进行有限的语法
```

func=lambda x,y:x+y
print(func(1,2))#输出3

reduce((lambda x,y:x+y),[1,2,3,4])#得到10

lower=(lambda x,y:x if x <y else y))
lower('bb','aa')#输出aa

```
##  列表解析表达式 
```

res=[c*4 for c in 'spam']#输出['ssss', 'pppp', 'aaaa', 'mmmm']
L=[x+10 for x in [1,2,3]]#得到[11,12,13]

f=open('text1.txt')
lines=[line.strip() for line in f]

lines=[line.strip() for line in f if line[0]=='p']

res=[x+y for x in [0,1,2] for y in [100,200,300]]

```

##  迭代器 
文件迭代器
```

f=open('text1.txt')
f.readline()
f.readline()
f.readline()
....
f=open('text1.txt')
f.__next__()
f.__next__()
f.__next__()
...
f=open('text1.txt')
next(f)
next(f)
next(f)
...

```
```

f=open('text1.txt')
for line in f
	print(line)
for line in f.readlines()
	print(line)

```
```

L=[1,2,3]
I=iter(L)
print(next(I))
print(next(I))
print(next(I))

```
##  生成器与yield 
Python对延迟提供了很多支持-它提供了工具在需要时候才产生结果，而不是立即产生结果。生成器创建时会自动实现迭代协议。
特别地，有两种语言结构尽可能地延迟结果创建。
  * 生成器函数:编写常规的def语句，但是使用yield返回一个结果，在每个结果之间挂起和继续它们的状态。通过迭代操作不断获取值
  * 生成器表达式
状态挂起与yield。迭代协议整合。StopIteration异常。__next__,next()
###  生成器函数 
```

def gen(N):
	for i in range(N):
		yield i**2

for i in gen(5):
	print(i,end=':')
输出:0:1:4:9:16

```
###  生成器表达式 
```

>>> G=(c*4 for c in 'SPAM')
>>> G
<generator object <genexpr> at 0x0000000003408BF8>
>>> list(G)
['SSSS', 'PPPP', 'AAAA', 'MMMM']

```
