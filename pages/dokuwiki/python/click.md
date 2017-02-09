title: click 

#  click强大的命令行工具 
官网：http://click.pocoo.org/5/
支持：
  * 命令的任意嵌套
  * 自动生成帮助信息
  * 支持在运行时子命令的延迟加载
安装方法是使用 pip：
pip install click

```

import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()

```

And what it looks like when run:

```

$ python hello.py --count=3

```
Your name: John
Hello John!
Hello John!
Hello John!
It automatically generates nicely formatted help pages:
```

$ python hello.py --help
Usage: hello.py [OPTIONS]
 Simple program that greets NAME for a total of COUNT times.
Options:
  --count INTEGER  Number of greetings.
  --name TEXT      The person to greet.
  --help           Show this message and exit


```
