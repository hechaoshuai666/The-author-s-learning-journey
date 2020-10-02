# 揭秘 Python 协程

## 什么是协程

用笔者个人的大粗话解释就是单线程,多任务

## 从一个爬虫说起

话不多说，我们先看一个简单的爬虫例子：

```python
import time
 
def crawl_page(url):
    print('crawling {}'.format(url))
    sleep_time = int(url.split('_')[-1])
    time.sleep(sleep_time)
    print('OK {}'.format(url))
 
def main(urls):
    for url in urls:
        crawl_page(url)
 
%time main(['url_1', 'url_2', 'url_3', 'url_4'])
 
########## 输出 ##########
 
crawling url_1
OK url_1
crawling url_2
OK url_2
crawling url_3
OK url_3
crawling url_4
OK url_4
Wall time: 10 s
```

（注意：本节的主要目的是协程的基础概念，因此我们简化爬虫的 scrawl_page 函数为休眠数秒，休眠时间取决于 url 最后的那个数字。）

这是一个很简单的爬虫，main() 函数执行时，调取 crawl_page() 函数进行网络通信，经过若干秒等待后收到结果，然后执行下一个。

看起来很简单，但你仔细一算，它也占用了不少时间，五个页面分别用了 1 秒到 4 秒的时间，加起来一共用了 10 秒。这显然效率低下，该怎么优化呢？

于是，一个很简单的思路出现了——我们这种爬取操作，完全可以并发化。我们就来看看使用协程怎么写。

```python
import asyncio
 
async def crawl_page(url):
    print('crawling {}'.format(url))
    sleep_time = int(url.split('_')[-1])
    await asyncio.sleep(sleep_time)
    print('OK {}'.format(url))
 
async def main(urls):
    for url in urls:
        await crawl_page(url)
 
%time asyncio.run(main(['url_1', 'url_2', 'url_3', 'url_4']))
 
########## 输出 ##########
 
crawling url_1
OK url_1
crawling url_2
OK url_2
crawling url_3
OK url_3
crawling url_4
OK url_4
Wall time: 10 s
```

看到这段代码，你应该发现了，在 Python 3.7 以上版本中，使用协程写异步程序非常简单。

首先来看 import asyncio，这个库包含了大部分我们实现协程所需的魔法工具。

async 修饰词声明异步函数，于是，这里的 crawl_page 和 main 都变成了异步函数。而调用异步函数，我们便可得到一个协程对象（coroutine object）。

举个例子，如果你 `print(crawl_page(''))`，便会输出`<coroutine object crawl_page at 0x000002BEDF141148>`，提示你这是一个 Python 的协程对象，而并不会真正执行这个函数。

再来说说协程的执行。执行协程有多种方法，这里我介绍一下常用的三种。

首先，我们可以通过 await 来调用。await 执行的效果，和 Python 正常执行是一样的，也就是说程序会阻塞在这里，进入被调用的协程函数，执行完毕返回后再继续，而这也是 await 的字面意思。代码中 `await asyncio.sleep(sleep_time)` 会在这里休息若干秒，`await crawl_page(url)` 则会执行 crawl_page() 函数。

最后，我们需要 asyncio.run 来触发运行。asyncio.run 这个函数是 Python 3.7 之后才有的特性，可以让 Python 的协程接口变得非常简单，你不用去理会事件循环怎么定义和怎么使用的问题。一个非常好的编程规范是，asyncio.run(main()) 作为主程序的入口函数，在程序运行周期内，只调用一次 asyncio.run。

这样，你就大概看懂了协程是怎么用的吧。不妨试着跑一下代码，欸，怎么还是 10 秒？

10 秒就对了，还记得上面所说的，await 是同步调用，因此， crawl_page(url) 在当前的调用结束之前，是不会触发下一次调用的。于是，这个代码效果就和上面完全一样了，相当于我们用异步接口写了个同步代码。

现在又该怎么办呢？

其实很简单，也正是我接下来要讲的协程中的一个重要概念，任务（Task）。老规矩，先看代码。

```python
import asyncio
 
async def crawl_page(url):
    print('crawling {}'.format(url))
    sleep_time = int(url.split('_')[-1])
    await asyncio.sleep(sleep_time)
    print('OK {}'.format(url))
 
async def main(urls):
    tasks = [asyncio.create_task(crawl_page(url)) for url in urls]
    for task in tasks:
        await task
 
%time asyncio.run(main(['url_1', 'url_2', 'url_3', 'url_4']))
 
########## 输出 ##########
 
crawling url_1
crawling url_2
crawling url_3
crawling url_4
OK url_1
OK url_2
OK url_3
OK url_4
Wall time: 3.99 s
```

你可以看到，我们有了协程对象后，便可以通过 `asyncio.create_task` 来创建任务。任务创建后很快就会被调度执行，这样，我们的代码也不会阻塞在任务这里。所以，我们要等所有任务都结束才行，用`for task in tasks: await task` 即可。

这次，你就看到效果了吧，结果显示，运行总时长等于运行时间最长的爬虫。

当然，你也可以想一想，这里用多线程应该怎么写？而如果需要爬取的页面有上万个又该怎么办呢？再对比下协程的写法，谁更清晰自是一目了然。

其实，对于执行 tasks，还有另一种做法：

```python
import asyncio
 
async def crawl_page(url):
    print('crawling {}'.format(url))
    sleep_time = int(url.split('_')[-1])
    await asyncio.sleep(sleep_time)
    print('OK {}'.format(url))
 
async def main(urls):
    tasks = [asyncio.create_task(crawl_page(url)) for url in urls]
    await asyncio.gather(*tasks)
 
%time asyncio.run(main(['url_1', 'url_2', 'url_3', 'url_4']))
 
########## 输出 ##########
 
crawling url_1
crawling url_2
crawling url_3
crawling url_4
OK url_1
OK url_2
OK url_3
OK url_4
Wall time: 4.01 s
```

## 解密协程运行时

说了这么多，现在，我们不妨来深入代码底层看看。有了前面的知识做基础，你应该很容易理解这两段代码。

```python
import asyncio
 
async def worker_1():
    print('worker_1 start')
    await asyncio.sleep(1)
    print('worker_1 done')
 
async def worker_2():
    print('worker_2 start')
    await asyncio.sleep(2)
    print('worker_2 done')
 
async def main():
    print('before await')
    await worker_1()
    print('awaited worker_1')
    await worker_2()
    print('awaited worker_2')
 
%time asyncio.run(main())
 
########## 输出 ##########
 
before await
worker_1 start
worker_1 done
awaited worker_1
worker_2 start
worker_2 done
awaited worker_2
Wall time: 3 s
```

```python
import asyncio
 
async def worker_1():
    print('worker_1 start')
    await asyncio.sleep(1)
    print('worker_1 done')
 
async def worker_2():
    print('worker_2 start')
    await asyncio.sleep(2)
    print('worker_2 done')
 
async def main():
    task1 = asyncio.create_task(worker_1())
    task2 = asyncio.create_task(worker_2())
    print('before await')
    await task1
    print('awaited worker_1')
    await task2
    print('awaited worker_2')
 
%time asyncio.run(main())
 
########## 输出 ##########
 
before await
worker_1 start
worker_2 start
worker_1 done
awaited worker_1
worker_2 done
awaited worker_2
Wall time: 2.01 s
```

不过，第二个代码，到底发生了什么呢？为了让你更详细了解到协程和线程的具体区别，这里我详细地分析了整个过程。步骤有点多，别着急，我们慢慢来看。

1. `asyncio.run(main())`，程序进入 main() 函数，事件循环开启；
2. task1 和 task2 任务被创建，并进入事件循环等待运行；运行到 print，输出 `'before await'`；
3. await task1 执行，用户选择从当前的主任务中切出，事件调度器开始调度 worker_1；
4. worker_1 开始运行，运行 print 输出`'worker_1 start'`，然后运行到 `await asyncio.sleep(1)`， 从当前任务切出，事件调度器开始调度 worker_2；
5. worker_2 开始运行，运行 print 输出 `'worker_2 start'`，然后运行 `await asyncio.sleep(2)` 从当前任务切出；
6. 以上所有事件的运行时间，都应该在 1ms 到 10ms 之间，甚至可能更短，事件调度器从这个时候开始暂停调度；
7. 一秒钟后，worker_1 的 sleep 完成，事件调度器将控制权重新传给 task_1，输出 `'worker_1 done'`，task_1 完成任务，从事件循环中退出；
8. await task1 完成，事件调度器将控制器传给主任务，输出 `'awaited worker_1'`，·然后在 await task2 处继续等待；
9. 两秒钟后，worker_2 的 sleep 完成，事件调度器将控制权重新传给 task_2，输出 `'worker_2 done'`，task_2 完成任务，从事件循环中退出；
10. 主任务输出 `'awaited worker_2'`，协程全任务结束，事件循环结束。

接下来，我们进阶一下。如果我们想给某些协程任务限定运行时间，一旦超时就取消，又该怎么做呢？再进一步，如果某些协程运行时出现错误，又该怎么处理呢？同样的，来看代码。

```python
import asyncio
 
async def worker_1():
    await asyncio.sleep(1)
    return 1
 
async def worker_2():
    await asyncio.sleep(2)
    return 2 / 0
 
async def worker_3():
    await asyncio.sleep(3)
    return 3
 
async def main():
    task_1 = asyncio.create_task(worker_1())
    task_2 = asyncio.create_task(worker_2())
    task_3 = asyncio.create_task(worker_3())
 
    await asyncio.sleep(2)
    task_3.cancel()
 
    res = await asyncio.gather(task_1, task_2, task_3, return_exceptions=True)
    print(res)
 
%time asyncio.run(main())
 
########## 输出 ##########
 
[1, ZeroDivisionError('division by zero'), CancelledError()]
Wall time: 2 s
```

你可以看到，worker_1 正常运行，worker_2 运行中出现错误，worker_3 执行时间过长被我们 cancel 掉了，这些信息会全部体现在最终的返回结果 res 中。

不过要注意`return_exceptions=True`这行代码。如果不设置这个参数，错误就会完整地 throw 到我们这个执行层，从而需要 try except 来捕捉，这也就意味着其他还没被执行的任务会被全部取消掉。为了避免这个局面，我们将 return_exceptions 设置为 True 即可。

到这里，发现了没，线程能实现的，协程都能做到。那就让我们温习一下这些知识点，用协程来实现一个经典的生产者消费者模型吧。

```python
import asyncio
import random
 
async def consumer(queue, id):
    while True:
        val = await queue.get()
        print('{} get a val: {}'.format(id, val))
        await asyncio.sleep(1)
 
async def producer(queue, id):
    for i in range(5):
        val = random.randint(1, 10)
        await queue.put(val)
        print('{} put a val: {}'.format(id, val))
        await asyncio.sleep(1)
 
async def main():
    queue = asyncio.Queue()
 
    consumer_1 = asyncio.create_task(consumer(queue, 'consumer_1'))
    consumer_2 = asyncio.create_task(consumer(queue, 'consumer_2'))
 
    producer_1 = asyncio.create_task(producer(queue, 'producer_1'))
    producer_2 = asyncio.create_task(producer(queue, 'producer_2'))
 
    await asyncio.sleep(10)
    consumer_1.cancel()
    consumer_2.cancel()
    
    await asyncio.gather(consumer_1, consumer_2, producer_1, producer_2, return_exceptions=True)
 
%time asyncio.run(main())
 
########## 输出 ##########
 
producer_1 put a val: 5
producer_2 put a val: 3
consumer_1 get a val: 5
consumer_2 get a val: 3
producer_1 put a val: 1
producer_2 put a val: 3
consumer_2 get a val: 1
consumer_1 get a val: 3
producer_1 put a val: 6
producer_2 put a val: 10
consumer_1 get a val: 6
consumer_2 get a val: 10
producer_1 put a val: 4
producer_2 put a val: 5
consumer_2 get a val: 4
consumer_1 get a val: 5
producer_1 put a val: 2
producer_2 put a val: 8
consumer_1 get a val: 2
consumer_2 get a val: 8
Wall time: 10 s
```

## 总结

+ 用async标识的函数为协程函数,直接调用并不会运行,且函数体中如果有io操作,则用await标明

+ 在python3.7+,可直接用asyncio.run(主函数调用),相当于之前版本的

  ```python
  import asyncio
  
  async def main():
      pass
  
  loop = asyncio.get_event_loop()
  loop.run_until_complete(main())
  ```

+ 在单线程中,有一个事件循环对象,可以往里面不断添加循环事件,按照create_task的添加的顺序为主

+ await是同步调用,默认会阻塞直到任务完成,才会继续向下执行

+ 只有在主线程阻塞的时候,事件循环才会调用队列里的任务,所以要有阻塞操作才会启动协程

+ 启动协程的手段有很多种,这里介绍常用的几种

  + asyncio.wait(iterable):接受可等待对象的迭代器
  + asyncio.gather(**iterable):大致功能跟wait差不多,但功能比wait更强大
  + asyncio.as_completed(aws):根据任务完成的快慢返回结果,上面两个都是固定顺序的

+ 要取消任务,在gather中,如果使用 return_exceptions = True,则会在返回的结果中体现CancelledError,默认为False,协程则会将异常抛出到应用层,要我们自己实现try except

