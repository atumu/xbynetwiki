createAt:2016-12-31 14:55:38
author:xbynet
modifyAt:2016-12-31 16:12:15
location:data/Python数据分析之NumPy
title:Python数据分析学习笔记之NumPy

数据源码:http://www.packtpub.com/support
数据分析的脑图:http://www.xmind.net/m/WvfC/
相关库和软件:
NumPy:提供常用的数值数组和函数。
SciPy:科学计算库，对NumPy的功能进行了扩充.
matplotlib:基于NumPy的绘图库。
IPython(目前改名为Jupyter)为交互式计算提供了一个基础设施。

为什么需要numpy
Python中提供了list容器，可以当作数组使用。但列表中的元素可以是任何对象，**因此列表中保存的是对象的指针**，这样一来，为了保存一个简单的列表[1,2,3]。就需要三个指针和三个整数对象。对于数值运算来说，这种结构显然不够高效。
Python虽然也提供了array模块，但其只支持一维数组，不支持多维数组，也没有各种运算函数。因而不适合数值运算。
而NumPy的出现弥补了这些不足。

# NumPy数组
NumPy的数组中比较重要**ndarray**对象属性有：

* ndarray.ndim：数组的维数（即数组轴的个数），等于秩。最常见的为二维数组（矩阵）。
* ndarray.shape：数组的维度。为一个表示数组在每个维度上大小的整数元组。例如二维数组中，表示数组的“行数”和“列数”。ndarray.shape返回一个元组，这个元组的长度就是维度的数目，即ndim属性。
* ndarray.size：数组元素的总个数，等于shape属性中元组元素的乘积。
* ndarray.dtype：表示数组中元素类型的对象，可使用标准的Python类型创建或指定dtype。
* ndarray.itemsize：数组中每个元素的字节大小。例如，一个元素类型为float64的数组itemsiz属性值为8(float64占用64个bits，每个字节长度为8，所以64/8，占用8个字节），又如，一个元素类型为complex32的数组item属性为4（32/8）。
* ndarray.data：包含实际数组元素的缓冲区，由于一般通过数组的索引获取元素，所以通常不需要使用这个属性。

创建数组
　　先来介绍创建数组。创建数组的方法有很多。如可以使用**array函数、arange()函数**或从常规的Python列表和元组创造数组。所创建的数组类型由原序列中的元素类型推导而来。
```
>>> from numpy import *  
　　　  
>>> a = array( [2,3,4] )　　　  
>>> a  
    array([2, 3, 4])  
>>> a.dtype  
    dtype('int32')  
>>> b = array([1.2, 3.5, 5.1])　　　  
>>> b.dtype  
    dtype('float64')  
```
使用array函数创建时，参数必须是由方括号括起来的列表，而不能使用多个数值作为参数调用array。　　
可使用双重序列来表示二维的数组，三重序列表示三维数组，以此类推。
可以在创建时显式指定数组中元素的类型。
NumPy提供一个类似**arange**的函数返回一个数列形式的数组:
```
>>> arange(10, 30, 5)  
    array([10, 15, 20, 25])  
```
以10开始，差值为5的等差数列。该函数不仅接受整数，还接受浮点参数：　
```
>>> arange(0,2,0.5)  
    array([ 0. ,  0.5,  1. ,  1.5])  
```
当arange使用浮点数参数时，由于浮点数精度有限，通常无法预测获得的元素个数。因此，最好使用函数linspace去接收我们想要的元素个数来代替用range来指定步长。linespace用法如下。
```
>>> numpy.linspace(-1, 0, 5)  
        array([-1.  , -0.75, -0.5 , -0.25,  0.  ])  
```
数组中的元素是通过下标来访问的，可以通过方括号括起一个下标来访问数组中单一一个元素，也可以以切片的形式访问数组中多个元素。

# NumPy中的数据类型
对于科学计算来说，Python中自带的整型、浮点型和复数类型远远不够，因此NumPy中添加了许多数据类型。如下：
![03efad14-cf20-11e6-9b73-00163e2eed34.png](/data/upload/03efad14-cf20-11e6-9b73-00163e2eed34.png)

# 数组的操作
数组的算术运算是按元素逐个运算。数组运算后将创建包含运算结果的新数组。
```
>>> a= np.array([20,30,40,50])  
>>> b= np.arange( 4)  
>>> b  
array([0, 1, 2, 3])  
>>> c= a-b  
>>> c  
array([20, 29, 38, 47])  
>>> b**2  
array([0, 1, 4, 9])  
>>> 10*np.sin(a)  
array([ 9.12945251,-9.88031624, 7.4511316, -2.62374854])  
>>> a<35  
array([True, True, False, False], dtype=bool)  
```
与其他矩阵语言不同，NumPy中的乘法运算符`*`按元素逐个计算.矩阵乘法可以使用dot函数或创建矩阵对象实现
```
>>> A= np.array([[1,1],  
...[0,1]])  
>>> B= np.array([[2,0],  
...[3,4]])  
>>> A*B # 逐个元素相乘  
array([[2, 0],  
    　　 [0, 4]])  
>>> np.dot(A,B) # 矩阵相乘  
array([[5, 4],  
    　　 [3, 4]])  
```
有些操作符如`+=和*=`用来**更改已存在数组**而不创建一个新的数组。

# 索引，切片和迭代
一维 数组可以被索引、切片和迭代，就像 列表 和其它Python序列。
```
>>> a = arange(10)**3
>>> a
array([  0,   1,   8,  27,  64, 125, 216, 343, 512, 729])
>>> a[2]
>>> a[2:5]
array([ 8, 27, 64])
>>> a[:6:2] = -1000    # equivalent to a[0:6:2] = -1000; from start to position 6, exclusive, set every 2nd element to -1000
array([-1000,     1, -1000,    27, -1000,   125,   216,   343,   512,   729])
>>> a[ : :-1]                                 # reversed a
array([  729,   512,   343,   216,   125, -1000,    27, -1000,     1, -1000])
```
除了使用下标范围存取元素之外，NumPy还提供了两种存取元素的高级方法。
**使用整数序列**
当使用整数序列对数组元素进行存取时，将使用整数序列中的每个元素作为下标，整数序列可以是列表或者数组。`使用整数序列作为下标获得的数组不和原始数组共享数据空间。`
**多维 数组可以每个轴有一个索引。这些索引由一个逗号分割的元组给出。**
```
>>> x = np.arange(10,1,-1)
>>> x
array([10,  9,  8,  7,  6,  5,  4,  3,  2])
>>> x[[3, 3, 1, 8]] # 获取x中的下标为3, 3, 1, 8的4个元素，组成一个新的数组
array([7, 7, 9, 2])
>>> b = x[np.array([3,3,-3,8])]  #下标可以是负数
>>> b[2] = 100
>>> b
array([7, 7, 100, 2])
>>> x   # 由于b和x不共享数据空间，因此x中的值并没有改变
array([10,  9,  8,  7,  6,  5,  4,  3,  2])
>>> x[[3,5,1]] = -1, -2, -3 # 整数序列下标也可以用来修改元素的值
>>> x
array([10, -3,  8, -1,  6, -2,  4,  3,  2])
```
使用布尔数组
当使用布尔数组b作为下标存取数组x中的元素时，将收集数组x中所有在数组b中对应下标为True的元素。使用布尔数组作为下标获得的数组不和原始数组共享数据空间，注意这种方式只对应于布尔数组，不能使用布尔列表。
```
>>> x = np.arange(5,0,-1)
>>> x
array([5, 4, 3, 2, 1])
>>> x[np.array([True, False, True, False, False])]
>>> # 布尔数组中下标为0，2的元素为True，因此获取x中下标为0,2的元素
array([5, 3])
>>> x[[True, False, True, False, False]]
>>> # 如果是布尔列表，则把True当作1, False当作0，按照整数序列方式获取x中的元素
array([4, 5, 4, 5, 5])
>>> x[np.array([True, False, True, True])]
>>> # 布尔数组的长度不够时，不够的部分都当作False
array([5, 3, 2])
>>> x[np.array([True, False, True, True])] = -1, -2, -3
>>> # 布尔数组下标也可以用来修改元素
>>> x
array([-1,  4, -2, -3,  1])
```
布尔数组一般不是手工产生，而是使用布尔运算的ufunc函数产生

当少于轴数的索引被提供时，确失的索引被认为是整个**切片**：
```
>>> b[-1]                                  # the last row. Equivalent to b[-1,:]
array([40, 41, 42, 43])
```
b[i] 中括号中的表达式被当作 i 和一系列 : ，来代表剩下的轴。NumPy也允许你使用“点”像 b[i,...] 。
点 (…)代表许多产生一个完整的索引元组必要的分号。如果x是秩为5的数组(即它有5个轴)，那么:
x[1,2,…] 等同于 x[1,2,:,:,:]
```
>>> for row in b:
...         print row
...
[0 1 2 3]
[10 11 12 13]
[20 21 22 23]
[30 31 32 33]
[40 41 42 43]
```
然而，如果一个人想对每个数组中元素进行运算，我们可以使用**flat属性，该属性是数组元素的一个迭代器**:
```
>>> for element in b.flat:
...         print element,
...
0 1 2 3 10 11 12 13 20 21 22 23 30 31 32 33 40 41 42 43
```
# 数组转换
tolist(),astype()
```
import numpy as np

b = np.array([ 1.+1.j,  3.+2.j])
print "In: b"
print b
#Out: array([ 1.+1.j,  3.+2.j])

print "In: b.tolist()"
print b.tolist()
#Out: [(1+1j), (3+2j)]

print "In: b.tostring()"
print b.tostring()
#Out: '\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\x08@\x00\x00\x00\x00\x00\x00\x00@'

print "In: fromstring('\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\x08@\x00\x00\x00\x00\x00\x00\x00@', dtype=complex)"
print np.fromstring('\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\xf0?\x00\x00\x00\x00\x00\x00\x08@\x00\x00\x00\x00\x00\x00\x00@', dtype=complex)
#Out: array([ 1.+1.j,  3.+2.j]

print "In: fromstring('20:42:52',sep=':', dtype=int)"
print np.fromstring('20:42:52',sep=':', dtype=int)
#Out: array([20, 42, 52])

print "In: b"
print b
#Out: array([ 1.+1.j,  3.+2.j])

print "In: b.astype(int)"
print b.astype(int)
#/usr/local/bin/ipython:1: ComplexWarning: Casting complex values to real discards the imaginary part
#  #!/usr/bin/python
#Out: array([1, 3])

print "In: b.astype('complex')"
print b.astype('complex')
#Out: array([ 1.+1.j,  3.+2.j])
```

# 更改数组的形状
```
import numpy as np

print "In: b = arange(24).reshape(2,3,4)"
b = np.arange(24).reshape(2,3,4)

print "In: b"
print b
#Out: 
#array([[[ 0,  1,  2,  3],
#        [ 4,  5,  6,  7],
#        [ 8,  9, 10, 11]],
#
#       [[12, 13, 14, 15],
#        [16, 17, 18, 19],
#        [20, 21, 22, 23]]])

print "In: b.ravel()"
print b.ravel()
#Out: 
#array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16,
#       17, 18, 19, 20, 21, 22, 23])

print "In: b.flatten()"
print b.flatten()
#Out: 
#array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16,
#       17, 18, 19, 20, 21, 22, 23])

print "In: b.shape = (6,4)"
b.shape = (6,4)

print "In: b"
print b
#Out: 
#array([[ 0,  1,  2,  3],
#       [ 4,  5,  6,  7],
#       [ 8,  9, 10, 11],
#       [12, 13, 14, 15],
#       [16, 17, 18, 19],
#       [20, 21, 22, 23]])

print "In: b.transpose()"
print b.transpose()
#Out: 
#array([[ 0,  4,  8, 12, 16, 20],
#       [ 1,  5,  9, 13, 17, 21],
#       [ 2,  6, 10, 14, 18, 22],
#       [ 3,  7, 11, 15, 19, 23]])


print "In: b.resize((2,12))"
b.resize((2,12))

print "In: b"
print b
#Out: 
#array([[ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11],
#       [12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]])
```

* revel()拆解：将多维数组变成一维数组。返回数组的视图
* flattern()拉直：功能与revel()相同，区别是，flattern返回的是真实的数组，需要分配新的内存空间，而revel只是返回数组的视图。
* reshape()：改变数组的形状。
* resize():功能类似于reshape(),但是resize会改变所作用的数组本身。
* transpose()：转置。对于二维表而言，转置就意味着行列交换。

# 组合(stack)不同的数组
几种方法可以沿不同轴将数组堆叠在一起：
```
import numpy as np

print "In: a = arange(9).reshape(3,3)"
a = np.arange(9).reshape(3,3)

print "In: a"
print a
#Out: 
#array([[0, 1, 2],
#       [3, 4, 5],
#       [6, 7, 8]])

print "In: b = 2 * a"
b = 2 * a

print "In: b"
print b
#Out: 
#array([[ 0,  2,  4],
#       [ 6,  8, 10],
#       [12, 14, 16]])

print "In: hstack((a, b))"
print np.hstack((a, b))
#Out: 
#array([[ 0,  1,  2,  0,  2,  4],
#       [ 3,  4,  5,  6,  8, 10],
#       [ 6,  7,  8, 12, 14, 16]])

print "In: concatenate((a, b), axis=1)"
print np.concatenate((a, b), axis=1)
#Out: 
#array([[ 0,  1,  2,  0,  2,  4],
#       [ 3,  4,  5,  6,  8, 10],
#       [ 6,  7,  8, 12, 14, 16]])

print "In: vstack((a, b))"
print np.vstack((a, b))
#Out: 
#array([[ 0,  1,  2],
#       [ 3,  4,  5],
#       [ 6,  7,  8],
#       [ 0,  2,  4],
#       [ 6,  8, 10],
#       [12, 14, 16]])

print "In: concatenate((a, b), axis=0)"
print np.concatenate((a, b), axis=0)
#Out: 
#array([[ 0,  1,  2],
#       [ 3,  4,  5],
#       [ 6,  7,  8],
#       [ 0,  2,  4],
#       [ 6,  8, 10],
#       [12, 14, 16]])

print "In: dstack((a, b))"
print np.dstack((a, b))
#Out: 
#array([[[ 0,  0],
#        [ 1,  2],
#        [ 2,  4]],
#
#       [[ 3,  6],
#        [ 4,  8],
#        [ 5, 10]],
#
#       [[ 6, 12],
#        [ 7, 14],
#        [ 8, 16]]])

print "In: oned = arange(2)"
oned = np.arange(2)

print "In: oned"
print oned
#Out: array([0, 1])

print "In: twice_oned = 2 * oned"
twice_oned = 2 * oned

print "In: twice_oned"
print twice_oned
#Out: array([0, 2])

print "In: column_stack((oned, twice_oned))"
print np.column_stack((oned, twice_oned)) 
#Out: 
#array([[0, 0],
#       [1, 2]])

print "In: column_stack((a, b))"
print np.column_stack((a, b))
#Out: 
#array([[ 0,  1,  2,  0,  2,  4],
#       [ 3,  4,  5,  6,  8, 10],
#       [ 6,  7,  8, 12, 14, 16]])

print "In: column_stack((a, b)) == hstack((a, b))"
print np.column_stack((a, b)) == np.hstack((a, b))
#Out: 
#array([[ True,  True,  True,  True,  True,  True],
#       [ True,  True,  True,  True,  True,  True],
#       [ True,  True,  True,  True,  True,  True]], dtype=bool)

print "In: row_stack((oned, twice_oned))"
print np.row_stack((oned, twice_oned))
#Out: 
#array([[0, 1],
#       [0, 2]])
 
print "In: row_stack((a, b))"
print np.row_stack((a, b))
#Out: 
#array([[ 0,  1,  2],
#       [ 3,  4,  5],
#       [ 6,  7,  8],
#       [ 0,  2,  4],
#       [ 6,  8, 10],
#       [12, 14, 16]])

print "In: row_stack((a,b)) == vstack((a, b))"
print np.row_stack((a,b)) == np.vstack((a, b))
#Out: 
#array([[ True,  True,  True],
#       [ True,  True,  True],
#       [ True,  True,  True],
#       [ True,  True,  True],
#       [ True,  True,  True],
#       [ True,  True,  True]], dtype=bool)
```
函数 column_stack 以列将一维数组合成二维数组，它等同与 vstack 对一维数组。
row_stack 函数，另一方面，将一维数组以行组合成二维数组。
对那些维度比二维更高的数组， hstack 沿着第二个轴组合， vstack 沿着第一个轴组合, concatenate 允许可选参数给出组合时沿着的轴。
在复杂情况下， r_[] 和 c_[] 对创建沿着一个方向组合的数很有用，它们允许范围符号(“:”):
```
>>> r_[1:4,0,4]
array([1, 2, 3, 0, 4])
```
当使用数组作为参数时， r_ 和 c_ 的默认行为和 vstack 和 hstack 很像，但是允许可选的参数给出组合所沿着的轴的代号。

# 将一个数组分割(split)成几个小数组
使用 hsplit 你能将数组沿着它的水平轴分割，或者指定返回相同形状数组的个数，或者指定在哪些列后发生分割，
vsplit 沿着纵向的轴分割， array split 允许指定沿哪个轴分割。

```
import numpy as np

print "In: a = arange(9).reshape(3, 3)"
a = np.arange(9).reshape(3, 3)

print "In: a"
print a
#Out: 
#array([[0, 1, 2],
#       [3, 4, 5],
#       [6, 7, 8]])

print "In: hsplit(a, 3)"
print np.hsplit(a, 3)
#Out: 
#[array([[0],
#       [3],
#       [6]]),
# array([[1],
#       [4],
#       [7]]),
# array([[2],
#       [5],
#       [8]])]

print "In: split(a, 3, axis=1)"
print np.split(a, 3, axis=1)
#Out: 
#[array([[0],
#       [3],
#       [6]]),
# array([[1],
#       [4],
#       [7]]),
# array([[2],
#       [5],
#       [8]])]

print "In: vsplit(a, 3)"
print np.vsplit(a, 3)
#Out: [array([[0, 1, 2]]), array([[3, 4, 5]]), array([[6, 7, 8]])]

print "In: split(a, 3, axis=0)"
print np.split(a, 3, axis=0)
#Out: [array([[0, 1, 2]]), array([[3, 4, 5]]), array([[6, 7, 8]])]

print "In: c = arange(27).reshape(3, 3, 3)"
c = np.arange(27).reshape(3, 3, 3)

print "In: c"
print c
#Out: 
#array([[[ 0,  1,  2],
#        [ 3,  4,  5],
#        [ 6,  7,  8]],
#
#       [[ 9, 10, 11],
#        [12, 13, 14],
#        [15, 16, 17]],
#
#       [[18, 19, 20],
#        [21, 22, 23],
#        [24, 25, 26]]])

print "In: dsplit(c, 3)"
print np.dsplit(c, 3)
#Out: 
#[array([[[ 0],
#        [ 3],
#        [ 6]],
#
#       [[ 9],
#        [12],
#        [15]],
#
#       [[18],
#        [21],
#        [24]]]),
# array([[[ 1],
#        [ 4],
#        [ 7]],
#
#       [[10],
#        [13],
#        [16]],
#
#       [[19],
#        [22],
#        [25]]]),
# array([[[ 2],
#        [ 5],
#        [ 8]],
#
#       [[11],
#        [14],
#        [17]],
#
#       [[20],
#        [23],
#        [26]]])]
```

# 复制和视图
**当运算和处理数组时，它们的数据有时被拷贝到新的数组有时不是**。这通常是新手的困惑之源。这有三种情况:
完全不拷贝
简单的赋值不拷贝数组对象或它们的数据。
```
>>> a = arange(12)
>>> b = a            # no new object is created
>>> b is a           # a and b are two names for the same ndarray object
True
>>> b.shape = 3,4    # changes the shape of a
>>> a.shape
(3, 4)
```
## 视图(view)和浅复制
注意：视图不是只读的。
**不同的数组对象分享同一个数据。视图方法创造一个新的数组对象指向同一数据。**
```
>>> c = a.view()
>>> c is a
False
>>> c.base is a                        # c is a view of the data owned by a
True
>>> c.flags.owndata
False
>>>
>>> c.shape = 2,6                      # a's shape doesn't change
>>> a.shape
(3, 4)
>>> c[0,4] = 1234                      # a's data changes
>>> a
array([[   0,    1,    2,    3],
       [1234,    5,    6,    7],
       [   8,    9,   10,   11]])
```
**切片数组返回它的一个视图**：
```
>>> s = a[ : , 1:3]     # spaces added for clarity; could also be written "s = a[:,1:3]"
>>> s[:] = 10           # s[:] is a view of s. Note the difference between s=10 and s[:]=10
>>> a
array([[   0,   10,   10,    3],
       [1234,   10,   10,    7],
       [   8,   10,   10,   11]])
```

## 深复制
这个复制方法完全复制数组和它的数据。
```
>>> d = a.copy()                          # a new array object with new data is created
>>> d is a
False
>>> d.base is a                           # d doesn't share anything with a
False
>>> d[0,0] = 9999
>>> a
array([[   0,   10,   10,    3],
       [1234,   10,   10,    7],
       [   8,   10,   10,   11]])
```

# 函数和方法(method)总览
这是个NumPy函数和方法分类排列目录。
创建数组
arange, array, copy, empty, empty_like, eye, fromfile, fromfunction, identity, linspace, logspace, mgrid, ogrid, ones, ones_like, r , zeros, zeros_like 
转化
astype, atleast 1d, atleast 2d, atleast 3d, mat 
操作
array split, column stack, concatenate, diagonal, dsplit, dstack, hsplit, hstack, item, newaxis, ravel, repeat, reshape, resize, squeeze, swapaxes, take, transpose, vsplit, vstack 
查询
all, any, nonzero, where 
排序
argmax, argmin, argsort, max, min, ptp, searchsorted, sort 
运算
choose, compress, cumprod, cumsum, inner, fill, imag, prod, put, putmask, real, sum 
基本统计
cov, mean, std, var 
基本线性代数
cross, dot, outer, svd, vdot

# 广播法则(rule)
广播法则能使通用函数有意义地处理不具有相同形状的输入。
广播第一法则是，如果所有的输入数组维度不都相同，一个“1”将被重复地添加在维度较小的数组上直至所有的数组拥有一样的维度。
广播第二法则确定长度为1的数组沿着特殊的方向表现地好像它有沿着那个方向最大形状的大小。对数组来说，沿着那个维度的数组元素的值理应相同。
应用广播法则之后，所有数组的大小必须匹配。
# 花哨的索引和索引技巧
NumPy比普通Python序列提供更多的索引功能。除了索引整数和切片，正如我们之前看到的，数组可以被整数数组和布尔数组索引。
```
通过数组索引
>>> a = arange(12)**2                          # the first 12 square numbers
>>> i = array( [ 1,1,3,8,5 ] )                 # an array of indices
>>> a[i]                                       # the elements of a at the positions i
array([ 1,  1,  9, 64, 25])
>>>
>>> j = array( [ [ 3, 4], [ 9, 7 ] ] )         # a bidimensional array of indices
>>> a[j]                                       # the same shape as j
array([[ 9, 16],
       [81, 49]])
```
通过布尔数组索引
当我们使用整数数组索引数组时，我们提供一个索引列表去选择。通过布尔数组索引的方法是不同的我们显式地选择数组中我们想要和不想要的元素。
我们能想到的使用布尔数组的索引最自然方式就是使用和原数组一样形状的布尔数组。

```
>>> a = arange(12).reshape(3,4)
>>> b = a > 4
>>> b                                          # b is a boolean with a's shape
array([[False, False, False, False],
       [False, True, True, True],
       [True, True, True, True]], dtype=bool)
>>> a[b]                                       # 1d array with the selected elements
array([ 5,  6,  7,  8,  9, 10, 11])
```

参考：
http://www.tuicool.com/articles/r2yyei
http://old.sebug.net/paper/books/scipydoc/numpy_intro.html
http://blog.csdn.net/sunny2038/article/details/9002531

