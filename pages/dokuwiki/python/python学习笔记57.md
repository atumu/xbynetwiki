title: python学习笔记57 

#  Python学习笔记之系统管理 
##  重定向脚本输入 

```

import fileinput
with fileinput.input as fin:
	for line in fin:
		print(line)


```
保存为filein.py.然后使用
```

./filein.py < /etc/passwd

```

##  终止程序并显示错误信息 
通常可以调用os.exit()
但是还有抛出异常方式. raise SystemExit("It fail!")

##  解析命令行选项 
使用argparse模块解析命令行选项。具体略。
```

# search.py
'''
Hypothetical command line tool for searching a collection of
files for one or more text patterns.
'''
import argparse
parser = argparse.ArgumentParser(description='Search some files')

parser.add_argument(dest='filenames',metavar='filename', nargs='*')

parser.add_argument('-p', '--pat',metavar='pattern', required=True,
                    dest='patterns', action='append',
                    help='text pattern to search for')

parser.add_argument('-v', dest='verbose', action='store_true', 
                    help='verbose mode')

parser.add_argument('-o', dest='outfile', action='store',
                    help='output file')

parser.add_argument('--speed', dest='speed', action='store',
                    choices={'slow','fast'}, default='slow',
                    help='search speed')

args = parser.parse_args()

# Output the collected arguments
print(args.filenames)
print(args.patterns)
print(args.verbose)
print(args.outfile)
print(args.speed)


```
##  在运行时提供密码输入提示 
```

import getpass

user = getpass.getuser()
passwd = getpass.getpass()

print('User:', user)
print('Passwd:', passwd)


```
##  执行外部命令并获取输出 
```

import subprocess
try:
    out_bytes = subprocess.check_output(['netstat', '-a']) #可选的参数shell=True,timeout=5
    out_text = out_bytes.decode('utf-8')
    print(out_text)
except subprocess.CalledProcessError as e:
    print('It did not work. Reason:', e)
    print('Exitcode:', e.returncode)

```
执行一个外部命令并获取输出，最简单的方法就是使用check_output函数了。但是如果需要同一个子进程执行更加高级的通信，例如为其发送输入，那就需要采用不同的方法了。基于此，可以使用subprocess.Popen类。如下:
```

import subprocess

# Some text to send
text = b'''
hello world
this is a test
goodbye
'''
# Launch a command with pipes
p = subprocess.Popen(['wc'],
          stdout = subprocess.PIPE,
          stdin = subprocess.PIPE)

# Send the data and get the output
stdout, stderr = p.communicate(text)

text = stdout.decode('utf-8')
print(text)


```
如果某个外部命令期望同一个真正的TTY（即终端设备）进行交互，比如sudo。那么subprocess就不能胜任了。这个时候我们可以选择pexpect模块或者其他渠道。
不过可以参考http://stackoverflow.com/questions/13045593/using-sudo-with-python-script
使用下面的方式也可以。
```

sp = Popen(cmd , shell=True, stdin=PIPE)
out, err = sp.communicate(_user_pass+'\n')   

```
