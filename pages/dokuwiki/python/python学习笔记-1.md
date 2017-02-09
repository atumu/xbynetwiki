title: python学习笔记-1 

#  Python程序的执行原理 
1. 过程概述
Python先把代码（.py文件）编译成字节码，交给字节码虚拟机，然后虚拟机一条一条执行字节码指令，从而完成程序的执行。

2. 字节码
字节码在Python虚拟机程序里对应的是` PyCodeObject `对象。 ` .pyc `文件是字节码在磁盘上的表现形式。

3. **pyc文件**
**PyCodeObject对象的创建时机是模块加载的时候，即import。**
Python test.py会对test.py进行编译成字节码并解释执行，但是不会生成test.pyc。
如果test.py加载了其他模块，如import util，Python会对util.py进行编译成字节码，生成util.pyc，然后对字节码解释执行。
如果想生成test.pyc，我们可以使用Python内置模块py_compile来编译。
加载模块时，如果同时存在.py和.pyc，Python会尝试使用.pyc，如果.pyc的编译时间早于.py的修改时间，则重新编译.py并更新.pyc。

##  4. PyCodeObject 
Python代码的编译结果就是PyCodeObject对象。
```

typedef struct {
    PyObject_HEAD
    int co_argcount;        /* 位置参数个数 */
    int co_nlocals;         /* 局部变量个数 */
    int co_stacksize;       /* 栈大小 */
    int co_flags;   
    PyObject *co_code;      /* 字节码指令序列 */
    PyObject *co_consts;    /* 所有常量集合 */
    PyObject *co_names;     /* 所有符号名称集合 */
    PyObject *co_varnames;  /* 局部变量名称集合 */
    PyObject *co_freevars;  /* 闭包用的的变量名集合 */
    PyObject *co_cellvars;  /* 内部嵌套函数引用的变量名集合 */
    /* The rest doesn’t count for hash/cmp */
    PyObject *co_filename;  /* 代码所在文件名 */
    PyObject *co_name;      /* 模块名|函数名|类名 */
    int co_firstlineno;     /* 代码块在文件中的起始行号 */
    PyObject *co_lnotab;    /* 字节码指令和行号的对应关系 */
    void *co_zombieframe;   /* for optimization only (see frameobject.c) */
} PyCodeObject;

```

##  5. pyc文件格式 
加载模块时，模块对应的PyCodeObject对象被写入.pyc文件，格式如下：
![](/data/dokuwiki/python/pasted/20160331-100029.png)

##  6. 分析字节码 
6.1 解析PyCodeObject
Python提供了内置函数compile可以编译Python代码和查看PyCodeObject对象，如下：
Python代码[test.py]
```

s = ”hello”
 
def func():
    print s
 
func()

```
在Python交互式shell里编译代码得到PyCodeObject对象:
![](/data/dokuwiki/python/pasted/20160331-100131.png)
dir(co)已经列出co的各个域，想查看某个域直接在终端输出即可：
![](/data/dokuwiki/python/pasted/20160331-100150.png)

**test.py的PyCodeObject:**
```

co.co_argcount    0
co.co_nlocals     0
co.co_names       (‘s’, ’func’)
co.co_varnames    (‘s’, ’func’)
co.co_consts      (‘hello’, <code object func at 0x2aaeeec57110, file ”test.py”, line 3>, None)
co.co_code        ’d\x00\x00Z\x00\x00d\x01\x00\x84\x00\x00Z\x01\x00e\x01\x00\x83\x00\x00\x01d\x02\x00S’

```

Python解释器会为函数也生成的字节码PyCodeObject对象，见上面的co_consts[1]
func的PyCodeObject
```

func.co_argcount   0
func.co_nlocals    0
func.co_names      (‘s’,)
func.co_varnames   ()
func.co_consts     (None,)
func.co_code       ‘t\x00\x00GHd\x00\x00S’

```
co_code是指令序列，是一串二进制流，它的格式和解析方法见6.2。

6.2 解析指令序列
指令序列co_code的格式
opcode	oparg	opcode	opcode	oparg	…
1 byte	2 bytes	1 byte	1 byte	2 bytes	
Python内置的dis模块可以解析co_code，如下图：
test.py的指令序列
![](/data/dokuwiki/python/pasted/20160331-100420.png)
func函数的指令序列
![](/data/dokuwiki/python/pasted/20160331-100433.png)
  * 第一列表示以下几个指令在py文件中的行号;
  * 第二列是该指令在指令序列co_code里的偏移量;
  * 第三列是指令opcode的名称，分为有操作数和无操作数两种，opcode在指令序列中是一个字节的整数;
  * 第四列是操作数oparg，在指令序列中占两个字节，基本都是co_consts或者co_names的下标;
  * 第五列带括号的是操作数说明。

##  7. 执行字节码 

Python虚拟机的原理就是模拟可执行程序再X86机器上的运行，X86的运行时栈帧如下图：
![](/data/dokuwiki/python/pasted/20160331-100521.png)
**当发生函数调用时，创建新的栈帧，对应Python的实现就是PyFrameObject对象。**

7.1 PyFrameObject
```

typedef struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;    /* 调用者的帧 */
    PyCodeObject *f_code;     /* 帧对应的字节码对象 */
    PyObject *f_builtins;     /* 内置名字空间 */
    PyObject *f_globals;      /* 全局名字空间 */
    PyObject *f_locals;       /* 本地名字空间 */
    PyObject **f_valuestack;  /* 运行时栈底 */
    PyObject **f_stacktop;    /* 运行时栈顶 */
    …….
}

```
7.2 执行指令
执行test.py的字节码时，会先创建一个栈帧，以下用f表示当前栈帧，执行过程注释如下：
test.py的符号名集合和常量集合
```

co.co_names   (‘s’, ’func’)
co.co_consts  (‘hello’, <code object func at 0x2aaeeec57110, file ”test.py”, line 3>, None)

```
test.py的指令序列
![](/data/dokuwiki/python/pasted/20160331-100744.png)
**上面的CALL_FUNCTION指令执行时，会创建新的栈帧，并执行func的字节码指令**，以下用f表示当前栈帧，func的字节码执行过程如下：
func函数的符号名集合和常量集合
```

func.co_names       (‘s’,)
func.co_consts      (None,)

```
func函数的指令序列
![](/data/dokuwiki/python/pasted/20160331-100845.png)
7.3 查看栈帧
如果你想查看当前栈帧，Python提供了` sys._getframe() `方法可以**获取当前栈帧**，你只需要在代码里加入代码如下：
```

def func():
    import sys
    frame = sys._getframe()
    print frame.f_locals
    print frame.f_globals
    print frame.f_back.f_locals
    #你可以打印frame的各个域
    print s

```
参考:http://python.jobbole.com/84599/