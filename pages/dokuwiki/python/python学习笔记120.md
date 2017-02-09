title: python学习笔记120 

#  Python总结之基础篇与内建&基础模块 
##  基础 
	查看帮助 help(),dir() ,__doc__
		import os
		for i in dir(os):
			print i         # 模块的方法
		help(os.path)       # 方法的帮助

	调试
		python -m trace -t aaaaaa.py
	
	pip模块安装
		
		yum install python-pip            # centos安装pip
		sudo apt-get install python-pip   # ubuntu安装pip
		pip官方安装脚本
			wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
			python get-pip.py
		加载环境变量
			vim /etc/profile
			export PATH=/usr/local/python27/bin:$PATH
			. /etc/profile

		pip install Package             # 安装包 pip install requests
		pip show --files Package        # 查看安装包时安装了哪些文件
		pip show --files Package        # 查看哪些包有更新
		pip install --upgrade Package   # 更新一个软件包
		pip uninstall Package           # 卸载软件包
		for循环

			sorted()           # 返回一个序列(列表)
			zip()              # 返回一个序列(列表)
			enumerate()        # 返回循环列表序列 for i,v in enumerate(['a','b']):
			reversed()         # 反序迭代器对象
			dict.keys()    # 通过键迭代
			dict.values()  # 通过值迭代
			dict.items()   # 通过键-值对迭代
			iter(obj)          # 得到obj迭代器 检查obj是不是一个序列
			iter(a,b)          # 重复调用a,直到迭代器的下一个值等于b
			for i in range(1, 5):
				print i
			else:
				print 'over'

			list = ['a','b','c','b']
			for i in range(len(list)):
				print list[i]
			for x, Lee in enumerate(list):
				print "%d %s Lee" % (x+1,Lee)
			
			# enumerate 使用函数得到索引值和对应值
			for i, v in enumerate(['tic', 'tac', 'toe']):
				print(i, v)

		流程结构简写

			[ i * 2 for i in [8,-2,5]]
			[16,-4,10]
			[ i for i in range(8) if i %2 == 0 ]
			[0,2,4,6]

##  inspect执行模块类中的所有方法 
```

# moniItems.py
import sys, time
import inspect

class mon:
    def __init__(self, n):
        self.name = n
        self.data = dict()
    def run(self):
        print('hello', self.name)
        return self.runAllGet()
    def getDisk(self):
        return 222
    def getCpu(self):
        return 111
    def runAllGet(self):
        for fun in inspect.getmembers(self, predicate=inspect.ismethod):
            print(fun[0], fun[1])
            if fun[0][:3] == 'get':
                    self.data[fun[0][3:]] = fun[1]()
        print(self.data)
        return self.data

m = mon('xxx')
m.runAllGet()

```
##  文件处理 
# 模式: 读'r'  写[清空整个文件]'w' 追加[文件需要存在]'a' 读写'r+' 二进制文件'b'  'rb','wb','rb+'

		写文件
			i={'ddd':'ccc'}
			f = file('poem.txt', 'a') 
			f.write("string")
			f.write(str(i))
			f.flush()
			f.close()

		读文件
			f = file('/etc/passwd','r')
			c = f.read().strip()        # 读取为一个大字符串，并去掉最后一个换行符
			for i in c.spilt('\n'):     # 用换行符切割字符串得到列表循环每行
				print i
			f.close()

		读文件1
			f = file('/etc/passwd','r')
			while True:
				line = f.readline()    # 返回一行
				if len(line) == 0:
					break
				x = line.split(":")                  # 冒号分割定义序列
				#x = [ x for x in line.split(":") ]  # 冒号分割定义序列
				#x = [ x.split("/") for x in line.split(":") ]  # 先冒号分割,在/分割 打印x[6][1]
				print x[6],"\n",
			f.close() 
		
		读文件2
			f = file('/etc/passwd')
			c = f.readlines()       # 读入所有文件内容,可反复读取,大文件时占用内存较大
			for line in c:
				print line.rstrip(),
			f.close()

		读文件3
			for i in open('b.txt'):   # 直接读取也可迭代,并有利于大文件读取,但不可反复读取
				print i,
		
		追加日志
			log = open('/home/peterli/xuesong','a')
			print >> log,'faaa'
			log.close()
		
		with读文件
			with open('a.txt') as f:
				for i in f:
					print i
				print f.read()        # 打印所有内容为字符串
				print f.readlines()   # 打印所有内容按行分割的列表
		
		csv读配置文件  
			192.168.1.5,web # 配置文件按逗号分割
			list = csv.reader(file('a.txt'))
			for line in list:
				print line              #  ['192.168.1.5', 'web']
##  内建函数 
		dir(sys)            # 显示对象的属性
		help(sys)           # 交互式帮助
		int(obj)            # 转型为整形
		str(obj)            # 转为字符串
		len(obj)            # 返回对象或序列长度
		open(file,mode)     # 打开文件 #mode (r 读,w 写, a追加)
		range(0,3)          # 返回一个整形列表，不包括3
		input("str:")   # 等待用户输入
		type(obj)           # 返回对象类型
		isinstance(object,int)    # 测试对象类型 int 
		abs(-22)            # 绝对值
		round(x[,n])        # 函数返回浮点数x的四舍五入值，如给出n值，则代表舍入到小数点后的位数
		del                 # 删除列表里面的数据
		max()               # 字符串中最大的字符
		min()               # 字符串中最小的字符
		sorted()            # 对序列排序
		reversed()          # 对序列倒序
		enumerate()         # 返回索引位置和对应的值
		sum()               # 总和
		list()              # 变成列表可用于迭代
		eval('3+4')         # 将字符串当表达式求值 得到7
		exec 'a=100'        # 将字符串按python语句执行
		tuple()             # 变成元组可用于迭代   #一旦初始化便不能更改的数据结构,速度比list快
		zip(s,t)            # 返回一个合并后的列表  s = ['11','22']  t = ['aa','bb']  [('11', 'aa'), ('22', 'bb')]
###  列表类型内建函数 
		list.append(obj)                 # 向列表中添加一个对象obj
		list.count(obj)                  # 返回一个对象obj在列表中出现的次数
		list.extend(seq)                 # 把序列seq的内容添加到列表中
		list.index(obj,i=0,j=len(list))  # 返回list[k] == obj 的k值,并且k的范围在i<=k<j;否则异常
		list.insert(index.obj)           # 在索引量为index的位置插入对象obj
		list.pop(index=-1)               # 删除并返回指定位置的对象,默认是最后一个对象
		list.remove(obj)                 # 从列表中删除对象obj
		list.reverse()                   # 原地翻转列表
		list.sort(func=None,key=None,reverse=False)  # 以指定的方式排序列表中成员,如果func和key参数指定,则按照指定的方式比较各个元素,如果reverse标志被置为True,则列表以反序排列

###  序列类型操作符 
		seq[ind]              # 获取下标为ind的元素
		seq[ind1:ind2]        # 获得下标从ind1到ind2的元素集合
		seq * expr            # 序列重复expr次
		seq1 + seq2           # 连接seq1和seq2
		obj in seq            # 判断obj元素是否包含在seq中
		obj not in seq        # 判断obj元素是否不包含在seq中
###  字符串类型内建方法 
		string.expandtabs(tabsize=8)                  # tab符号转为空格 #默认8个空格
		string.endswith(obj,beg=0,end=len(staring))   # 检测字符串是否已obj结束,如果是返回True #如果beg或end指定检测范围是否已obj结束
		string.count(str,beg=0,end=len(string))       # 检测str在string里出现次数  f.count('\n',0,len(f)) 判断文件行数
		string.find(str,beg=0,end=len(string))        # 检测str是否包含在string中
		string.index(str,beg=0,end=len(string))       # 检测str不在string中,会报异常
		string.isalnum()                              # 如果string至少有一个字符并且所有字符都是字母或数字则返回True
		string.isalpha()                              # 如果string至少有一个字符并且所有字符都是字母则返回True
		string.isnumeric()                            # 如果string只包含数字字符,则返回True
		string.isspace()                              # 如果string包含空格则返回True
		string.isupper()                              # 字符串都是大写返回True
		string.islower()                              # 字符串都是小写返回True
		string.lower()                                # 转换字符串中所有大写为小写
		string.upper()                                # 转换字符串中所有小写为大写
		string.lstrip()                               # 去掉string左边的空格
		string.rstrip()                               # 去掉string字符末尾的空格
		string.replace(str1,str2,num=string.count(str1))  # 把string中的str1替换成str2,如果num指定,则替换不超过num次
		string.startswith(obj,beg=0,end=len(string))  # 检测字符串是否以obj开头
		string.zfill(width)                           # 返回字符长度为width的字符,原字符串右对齐,前面填充0
		string.isdigit()                              # 只包含数字返回True
		string.split("分隔符")                        # 把string切片成一个列表
		":".join(string.split())                      # 以:作为分隔符,将所有元素合并为一个新的字符串
###  字典内建方法 
		dict.clear()                            # 删除字典中所有元素
		dict copy()                             # 返回字典(浅复制)的一个副本
		dict.fromkeys(seq,val=None)             # 创建并返回一个新字典,以seq中的元素做该字典的键,val做该字典中所有键对的初始值
		dict.get(key,default=None)              # 对字典dict中的键key,返回它对应的值value,如果字典中不存在此键,则返回default值
		dict.has_key(key)                       # 如果键在字典中存在,则返回True 用in和not in代替
		dicr.items()                            # 返回一个包含字典中键、值对元组的列表
		dict.keys()                             # 返回一个包含字典中键的列表
		dict.iter()                             # 方法iteritems()、iterkeys()、itervalues()与它们对应的非迭代方法一样,不同的是它们返回一个迭代子,而不是一个列表
		dict.pop(key[,default])                 # 和方法get()相似.如果字典中key键存在,删除并返回dict[key]
		dict.setdefault(key,default=None)       # 和set()相似,但如果字典中不存在key键,由dict[key]=default为它赋值
		dict.update(dict2)                      # 将字典dict2的键值对添加到字典dict
		dict.values()                           # 返回一个包含字典中所有值得列表
		dict([container])     # 创建字典的工厂函数。提供容器类(container),就用其中的条目填充字典
		len(mapping)          # 返回映射的长度(键-值对的个数)
		hash(obj)             # 返回obj哈希值,判断某个对象是否可做一个字典的键值		
		
###  集合方法 
		s.update(t)                         # 用t中的元素修改s,s现在包含s或t的成员   s |= t
		s.intersection_update(t)            # s中的成员是共用属于s和t的元素          s &= t
		s.difference_update(t)              # s中的成员是属于s但不包含在t中的元素    s -= t
		s.symmetric_difference_update(t)    # s中的成员更新为那些包含在s或t中,但不是s和t共有的元素  s ^= t
		s.add(obj)                          # 在集合s中添加对象obj
		s.remove(obj)                       # 从集合s中删除对象obj;如果obj不是集合s中的元素(obj not in s),将引发KeyError错误
		s.discard(obj)                      # 如果obj是集合s中的元素,从集合s中删除对象obj
		s.pop()                             # 删除集合s中的任意一个对象,并返回它
		s.clear()                           # 删除集合s中的所有元素
		s.issubset(t)                       # 如果s是t的子集,则返回True   s <= t
		s.issuperset(t)                     # 如果t是s的超集,则返回True   s >= t
		s.union(t)                          # 合并操作;返回一个新集合,该集合是s和t的并集   s | t
		s.intersection(t)                   # 交集操作;返回一个新集合,该集合是s和t的交集   s & t
		s.difference(t)                     # 返回一个新集合,改集合是s的成员,但不是t的成员  s - t
		s.symmetric_difference(t)           # 返回一个新集合,该集合是s或t的成员,但不是s和t共有的成员   s ^ t
		s.copy()                            # 返回一个新集合,它是集合s的浅复制
		obj in s                            # 成员测试;obj是s中的元素 返回True
		obj not in s                        # 非成员测试:obj不是s中元素 返回True
		s == t                              # 等价测试 是否具有相同元素
		s != t                              # 不等价测试 
		s < t                               # 子集测试;s!=t且s中所有元素都是t的成员
		s > t                               # 超集测试;s!=t且t中所有元素都是s的成员
###  文件对象方法 
		file.close()                     # 关闭文件
		file.fileno()                    # 返回文件的描述符
		file.flush()                     # 刷新文件的内部缓冲区
		file.isatty()                    # 判断file是否是一个类tty设备
		file.next()                      # 返回文件的下一行,或在没有其他行时引发StopIteration异常
		file.read(size=-1)               # 从文件读取size个字节,当未给定size或给定负值的时候,读取剩余的所有字节,然后作为字符串返回
		file.readline(size=-1)           # 从文件中读取并返回一行(包括行结束符),或返回最大size个字符
		file.readlines(sizhint=0)        # 读取文件的所有行作为一个列表返回
		file.xreadlines()                # 用于迭代,可替换readlines()的一个更高效的方法
		file.seek(off, whence=0)         # 在文件中移动文件指针,从whence(0代表文件起始,1代表当前位置,2代表文件末尾)偏移off字节
		file.tell()                      # 返回当前在文件中的位置
		file.truncate(size=file.tell())  # 截取文件到最大size字节,默认为当前文件位置
		file.write(str)                  # 向文件写入字符串
		file.writelines(seq)             # 向文件写入字符串序列seq;seq应该是一个返回字符串的可迭代对象
###  文件对象的属性 
		file.closed          # 表示文件已被关闭,否则为False
		file.encoding        # 文件所使用的编码  当unicode字符串被写入数据时,它将自动使用file.encoding转换为字节字符串;若file.encoding为None时使用系统默认编码
		file.mode            # Access文件打开时使用的访问模式
		file.name            # 文件名
		file.newlines        # 未读取到行分隔符时为None,只有一种行分隔符时为一个字符串,当文件有多种类型的行结束符时,则为一个包含所有当前所遇到的行结束符的列表
		file.softspace       # 为0表示在输出一数据后,要加上一个空格符,1表示不加
###  函数式编程的内建函数 
		apply(func[,nkw][,kw])          # 用可选的参数来调用func,nkw为非关键字参数,kw为关键字参数;返回值是函数调用的返回值 （Python3.x不建议使用）
		filter(func,seq)                # 调用一个布尔函数func来迭代遍历每个seq中的元素;返回一个使func返回值为true的元素的序列
		map(func,seq1[,seq2])           # 将函数func作用于给定序列(s)的每个元素,并用一个列表来提供返回值;如果func为None,func表现为一个身份函数,返回一个含有每个序列中元素集合的n个元组的列表
		reduce(func,seq[,init])         # 将二元函数作用于seq序列的元素,每次携带一堆(先前的结果以及下一个序列元素),连续地将现有的结果和下一个值作用在获得的随后的结果上,最后减少我们的序列为一个单一的返回值;如果初始值init给定,第一个比较会是init和第一个序列元素而不是序列的头两个元素
```

		# filter 即通过函数方法只保留结果为真的值组成列表
		def f(x): return x % 2 != 0 and x % 3 != 0
		f(3)     # 函数结果是False  3被filter抛弃
		f(5)     # 函数结果是True   5被加入filter最后的列表结果
		filter(f, range(2, 25))
		[5, 7, 11, 13, 17, 19, 23]
		
		# map 通过函数对列表进行处理得到新的列表
		def cube(x): return x*x*x
		map(cube, range(1, 11))
		[1, 8, 27, 64, 125, 216, 343, 512, 729, 1000]
		
		# reduce 通过函数会先接收初始值和序列的第一个元素，然后是返回值和下一个元素，依此类推
		def add(x,y): return x+y
		reduce(add, range(1, 11))      # 结果55  是1到10的和  x的值是上一次函数返回的结果，y是列表中循环的值

```


##  字符串相关模块 
		string         # 字符串操作相关函数和工具
		re             # 正则表达式
		struct         # 字符串和二进制之间的转换
		c/StringIO     # 字符串缓冲对象,操作方法类似于file对象
		base64         # Base16\32\64数据编解码
		codecs         # 解码器注册和基类
		crypt          # 进行单方面加密
		difflib        # 找出序列间的不同
		hashlib        # 多种不同安全哈希算法和信息摘要算法的API
		hma            # HMAC信息鉴权算法的python实现
		md5            # RSA的MD5信息摘要鉴权
		rotor          # 提供多平台的加解密服务
		sha            # NIAT的安全哈希算法SHA
		stringprep     # 提供用于IP协议的Unicode字符串
		textwrap       # 文本包装和填充
		unicodedate    # unicode数据库
##  序列类型相关的模块 
		array         # 一种受限制的可变序列类型,元素必须相同类型
		copy          # 提供浅拷贝和深拷贝的能力
		operator      # 包含函数调用形式的序列操作符 operator.concat(m,n)
		re            # perl风格的正则表达式查找
		StringIO      # 把长字符串作为文件来操作 如: read() \ seek()
		cStringIO     # 把长字符串作为文件来操,作速度更快,但不能被继承
		textwrap      # 用作包装/填充文本的函数,也有一个类
		types         # 包含python支持的所有类型
		collections   # 高性能容器数据类型
##  异常处理 
	
```

		# try 中使用 sys.exit(2) 会被捕获,无法退出脚本,可使用 os._exit(2) 退出脚本
		class ShortInputException(Exception):  # 继承Exception异常的类,定义自己的异常
			def __init__(self, length, atleast):
				super(Exception,self).__init__()
				self.length = length
				self.atleast = atleast
		try:
			s = input('Enter something --> ')
			if len(s) < 3:
				raise ShortInputException(len(s), 3)    # 触发异常
		except EOFError:
			print '\nWhy did you do an EOF on me?'
		except ShortInputException, x:      # 捕捉指定错误信息
			print 'ShortInputException:  %d | %d' % (x.length, x.atleast)
		except Exception as err:            # 捕捉所有其它错误信息内容
			print str(err)
		#except urllib2.HTTPError as err:   # 捕捉外部导入模块的错误
		#except:                            # 捕捉所有其它错误 不会看到错误内容
		#		print 'except'
		finally:                            # 无论什么情况都会执行 关闭文件或断开连接等
			   print 'finally' 
		else:                               # 无任何异常 无法和finally同用
			print 'No exception was raised.'

``` 
###  不可捕获的异常 
			NameError:              # 尝试访问一个未申明的变量
			ZeroDivisionError:      # 除数为零
			SyntaxErrot:            # 解释器语法错误
			IndexError:             # 请求的索引元素超出序列范围
			KeyError:               # 请求一个不存在的字典关键字
			IOError:                # 输入/输出错误
			AttributeError:         # 尝试访问未知的对象属性
			ImportError             # 没有模块
			IndentationError        # 语法缩进错误
			KeyboardInterrupt       # ctrl+C
			SyntaxError             # 代码语法错误
			ValueError              # 值错误
			TypeError               # 传入对象类型与要求不符合

###  内建异常 
			
			BaseException                # 所有异常的基类
			SystemExit                   # python解释器请求退出
			KeyboardInterrupt            # 用户中断执行
			Exception                    # 常规错误的基类
			StopIteration                # 迭代器没有更多的值
			GeneratorExit                # 生成器发生异常来通知退出
			StandardError                # 所有的内建标准异常的基类
			ArithmeticError              # 所有数值计算错误的基类
			FloatingPointError           # 浮点计算错误
			OverflowError                # 数值运算超出最大限制
			AssertionError               # 断言语句失败
			AttributeError               # 对象没有这个属性
			EOFError                     # 没有内建输入,到达EOF标记
			EnvironmentError             # 操作系统错误的基类
			IOError                      # 输入/输出操作失败
			OSError                      # 操作系统错误
			WindowsError                 # windows系统调用失败
			ImportError                  # 导入模块/对象失败
			KeyboardInterrupt            # 用户中断执行(通常是ctrl+c)
			LookupError                  # 无效数据查询的基类
			IndexError                   # 序列中没有此索引(index)
			KeyError                     # 映射中没有这个键
			MemoryError                  # 内存溢出错误(对于python解释器不是致命的)
			NameError                    # 未声明/初始化对象(没有属性)
			UnboundLocalError            # 访问未初始化的本地变量
			ReferenceError               # 若引用试图访问已经垃圾回收了的对象
			RuntimeError                 # 一般的运行时错误
			NotImplementedError          # 尚未实现的方法
			SyntaxError                  # python语法错误
			IndentationError             # 缩进错误
			TabError                     # tab和空格混用
			SystemError                  # 一般的解释器系统错误
			TypeError                    # 对类型无效的操作
			ValueError                   # 传入无效的参数
			UnicodeError                 # Unicode相关的错误
			UnicodeDecodeError           # Unicode解码时的错误
			UnicodeEncodeError           # Unicode编码时的错误
			UnicodeTranslateError        # Unicode转换时错误
			Warning                      # 警告的基类
			DeprecationWarning           # 关于被弃用的特征的警告
			FutureWarning                # 关于构造将来语义会有改变的警告
			OverflowWarning              # 旧的关于自动提升为长整形的警告
			PendingDeprecationWarning    # 关于特性将会被废弃的警告
			RuntimeWarning               # 可疑的运行时行为的警告
			SyntaxWarning                # 可疑的语法的警告
			UserWarning                  # 用户代码生成的警告
###  触发异常 
			raise exclass            # 触发异常,从exclass生成一个实例(不含任何异常参数)
			raise exclass()          # 触发异常,但现在不是类;通过函数调用操作符(function calloperator:"()")作用于类名生成一个新的exclass实例,同样也没有异常参数
			raise exclass, args      # 触发异常,但同时提供的异常参数args,可以是一个参数也可以是元组
			raise exclass(args)      # 触发异常,同上
			raise exclass, args, tb  # 触发异常,但提供一个跟踪记录(traceback)对象tb供使用
			raise exclass,instance   # 通过实例触发异常(通常是exclass的实例)
			raise instance           # 通过实例触发异常;异常类型是实例的类型:等价于raise instance.__class__, instance
			raise string             # 触发字符串异常
			raise string, srgs       # 触发字符串异常,但触发伴随着args
			raise string,args,tb     # 触发字符串异常,但提供一个跟踪记录(traceback)对象tb供使用
			raise                    # 重新触发前一个异常,如果之前没有异常,触发TypeError
###  跟踪异常栈 
```

# traceback 获取异常相关数据都是通过sys.exc_info()函数得到的
			import traceback
			import sys
			try:
				s = input()
				print int(s)
			except ValueError:
				# sys.exc_info() 返回值是元组，第一个exc_type是异常的对象类型，exc_value是异常的值，exc_tb是一个traceback对象，对象中包含出错的行数、位置等数据
				exc_type, exc_value, exc_tb = sys.exc_info()
				print "\n%s \n %s \n %s\n" %(exc_type, exc_value, exc_tb )
				traceback.print_exc()        # 打印栈跟踪信息

```
###  抓取全部错误信息存如字典 
```

import sys, traceback
			try:
				s = raw_input()
				int(s)
			except:
				exc_type, exc_value, exc_traceback = sys.exc_info() 
				traceback_details = {
									 'filename': exc_traceback.tb_frame.f_code.co_filename,
									 'lineno'  : exc_traceback.tb_lineno,
									 'name'    : exc_traceback.tb_frame.f_code.co_name,
									 'type'    : exc_type.__name__,
									 'message' : exc_value.message, 
									}
			 
				del(exc_type, exc_value, exc_traceback) 
				print traceback_details
				f = file('test1.txt', 'a')
				f.write("%s %s %s %s %s\n" %(traceback_details['filename'],traceback_details['lineno'],traceback_details['name'],traceback_details['type'],traceback_details['message'], ))
				f.flush()
				f.close()

```
调试log
```


		# cgitb覆盖了默认sys.excepthook全局异常拦截器
		def func(a, b):
			return a / b
		if __name__ == '__main__':
			import cgitb
			cgitb.enable(format='text')
			func(1, 0)

```
##  re正则表达式 
		compile(pattern,flags=0)          # 对正则表达式模式pattern进行编译,flags是可选标识符,并返回一个regex对象
		match(pattern,string,flags=0)     # 尝试用正则表达式模式pattern匹配字符串string,flags是可选标识符,如果匹配成功,则返回一个匹配对象;否则返回None
		search(pattern,string,flags=0)    # 在字符串string中搜索正则表达式模式pattern的第一次出现,flags是可选标识符,如果匹配成功,则返回一个匹配对象;否则返回None
		findall(pattern,string[,flags])   # 在字符串string中搜索正则表达式模式pattern的所有(非重复)出现:返回一个匹配对象的列表  # pattern=u'\u4e2d\u6587' 代表UNICODE
		finditer(pattern,string[,flags])  # 和findall()相同,但返回的不是列表而是迭代器;对于每个匹配,该迭代器返回一个匹配对象
		split(pattern,string,max=0)       # 根据正则表达式pattern中的分隔符把字符string分割为一个列表,返回成功匹配的列表,最多分割max次(默认所有)
		sub(pattern,repl,string,max=0)    # 把字符串string中所有匹配正则表达式pattern的地方替换成字符串repl,如果max的值没有给出,则对所有匹配的地方进行替换(subn()会返回一个表示替换次数的数值)
		group(num=0)                      # 返回全部匹配对象(或指定编号是num的子组)
		groups()                          # 返回一个包含全部匹配的子组的元组(如果没匹配成功,返回一个空元组)
		
```

		例子
			re.findall(r'a[be]c','123abc456eaec789')         # 返回匹配对象列表 ['abc', 'aec']
			re.findall("(.)12[34](..)",a)                    # 取出匹配括号中内容   a='qedqwe123dsf'
			re.search("(.)123",a ).group(1)                  # 搜索匹配的取第1个标签
			re.match("^(1|2) *(.*) *abc$", str).group(2)     # 取第二个标签
			re.match("^(1|2) *(.*) *abc$", str).groups()     # 取所有标签
			re.sub('[abc]','A','alex')                       # 替换
			for i in re.finditer(r'\d+',s):                  # 迭代
				print i.group(),i.span()                     #
		
		搜索网页中UNICODE格式的中文
			QueryAdd='http://www.anti-spam.org.cn/Rbl/Query/Result'
			Ip='222.129.184.52'
			s = requests.post(url=QueryAdd, data={'IP':Ip})
			re.findall(u'\u4e2d\u56fd', s.text, re.S)

```
                        
