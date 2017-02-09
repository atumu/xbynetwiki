title: python学习笔记55 

#  Python学习笔记之asyncio与requests结合 
To use requests (or any other blocking libraries) with asyncio, you can use BaseEventLoop.run_in_executor to run a function in another thread and yield from it to get the result. For example:
为了在requests或者任何其他的阻塞库中使用asyncio.你可以使用` BaseEventLoop.run_in_executor `去另一个线程执行函数，然后yield from等待所有结果。
BaseEventLoop.run_in_executor(executor, func, *args)。如果executor=None那么将使用默认的 Executor实例。Executor可以是(pool of threads or pool of processes)。默认使用ThreadPoolExecutor
##  Python3.4的版本 
```

import asyncio
import requests
@asyncio.coroutine
def main():
    loop = asyncio.get_event_loop()
    future1 = loop.run_in_executor(None, requests.get, 'http://www.google.com')
    future2 = loop.run_in_executor(None, requests.get, 'http://www.google.co.uk')
    response1 = yield from future1
    response2 = yield from future2
    print(response1.text)
    print(response2.text)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

```
##  Python3.5版本 
With python 3.5 you can use the new away/async syntax:
```

import asyncio
import requests
async def main():
    loop = asyncio.get_event_loop()
    future1 = loop.run_in_executor(None, requests.get, 'http://www.google.com')
    future2 = loop.run_in_executor(None, requests.get, 'http://www.google.co.uk')
    response1 = await future1
    response2 = await future2
    print(response1.text)
    print(response2.text)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

```
参考:http://stackoverflow.com/questions/22190403/how-could-i-use-requests-in-asyncio