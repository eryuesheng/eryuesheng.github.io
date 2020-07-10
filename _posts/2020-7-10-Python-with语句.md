---
layout:     post
title:      python with语句
subtitle:   Python with
date:       2020-7-10
author:     ErYueSheng
header-img: img/post-sample-image.jpg
catalog:    true
tags:
    - python
    - with语句
---

>学习至[浅谈 Python 的 with 语句](https://developer.ibm.com/zh/articles/os-cn-pythonwith/)

## 引言

with语句适用于对需要对资源进行访问的场合，确保不管因为什么情况发生异常都会执行清理操作，释放资源。如文件的自动打开关闭、线程锁的自动获取和释放。

## 术语

有了上下文管理器，with语句才能工作。上下文管理器的有关概念如下：

- 上下文管理协议（Context Management Protocol）：包含方法__enter__()和__exit__(),支持该协议的对象要实现这两个方法。

- 上下文管理器（Context Manager）：支持上下文管理协议的对象，这种对象实现了__enter__()和__exit__()方法。上下文管理器定义执行 with 语句时要建立的运行时上下文，负责执行 with 语句块上下文中的进入与退出操作。通常使用 with 语句调用上下文管理器，也可以通过直接调用其方法来使用。

- 运行时上下文（runtime context）：由上下文管理器创建，通过上下文管理器的__enter__()和__exit__()方法实现，\_\_enter__()方法在语句体执行之前进入运行时上下文，\_\_exit__()在语句体执行完后从运行时上下文退出。with 语句支持运行时上下文这一概念。

- 上下文表达式（Context Expression） ：with 语句中跟在关键字 with 之后的表达式，该表达式要返回一个上下文管理器对象。

- 语句体（with-body） ：with 语句包裹起来的代码块，在执行语句体之前会调用上下文管理器的 enter() 方法，执行完语句体之后会执行 exit() 方法。

## 基本语法和工作原理

>不想写了，而且也没有啥实例，然后就随便找了一个[别的](https://www.cnblogs.com/DswCnblog/p/6126588.html),复制了一通，草草完事。20200710

有一些任务，可能事先需要设置，事后做清理工作。对于这种场景，Python的with语句提供了一种非常方便的处理方式。一个很好的例子是文件处理，你需要获取一个文件句柄，从文件中读取数据，然后关闭文件句柄。

如果不用with语句，代码如下：

    file = open("/tmp/foo.txt")
    data = file.read()
    file.close()

这里有两个问题。一是可能忘记关闭文件句柄；二是文件读取数据发生异常，没有进行任何处理。下面是处理异常的加强版本：

    file = open("/tmp/foo.txt")
    try:
        data = file.read()
    finally:
        file.close()

虽然这段代码运行良好，但是太冗长了。这时候就是with一展身手的时候了。除了有更优雅的语法，with还可以很好的处理上下文环境产生的异常。下面是with版本的代码：

    with open("/tmp/foo.txt") as file:
        data = file.read()
    
**with如何工作？**

这看起来充满魔法，但不仅仅是魔法，Python对with的处理还很聪明。基本思想是with所求值的对象必须有一个__enter__()方法，一个__exit__()方法。

紧跟with后面的语句被求值后，返回对象的__enter__()方法被调用，这个方法的返回值将被赋值给as后面的变量。当with后面的代码块全部被执行完之后，将调用前面返回对象的__exit__()方法。

下面例子可以具体说明with如何工作：

    class Sample:
        def __enter__(self):
            print "In __enter__()"
            return "Foo"
    
        def __exit__(self, type, value, trace):
            print "In __exit__()"
    
    def get_sample():
        return Sample()
    
    with get_sample() as sample:
        print "sample:", sample

运行代码，输出如下

    In __enter__()
    sample: Foo
    In __exit__()

正如你看到的，

1. \_\_enter__()方法被执行
2. \_\_enter__()方法返回的值 - 这个例子中是"Foo"，赋值给变量'sample'
3. 执行代码块，打印变量"sample"的值为 "Foo"
4. \_\_exit__()方法被调用

with真正强大之处是它可以处理异常。可能你已经注意到Sample类的__exit__方法有三个参数- val, type 和 trace。 这些参数在异常处理中相当有用。我们来改一下代码，看看具体如何工作的。

    class Sample:
        def __enter__(self):
            return self
    
        def __exit__(self, type, value, trace):
            print "type:", type
            print "value:", value
            print "trace:", trace
    
        def do_something(self):
            bar = 1/0
            return bar + 10
    
    with Sample() as sample:
        sample.do_something()

这个例子中，with后面的get_sample()变成了Sample()。这没有任何关系，只要紧跟with后面的语句所返回的对象有__enter__()和__exit__()方法即可。此例中，Sample()的__enter__()方法返回新创建的Sample对象，并赋值给变量sample。

代码执行后：

    type: <type 'exceptions.ZeroDivisionError'>
    value: integer division or modulo by zero
    trace: <traceback object at 0x1004a8128>
    Traceback (most recent call last):
    File "./with_example02.py", line 19, in <module>
        sample.do_something()
    File "./with_example02.py", line 15, in do_something
        bar = 1/0
    ZeroDivisionError: integer division or modulo by zero

实际上，在with后面的代码块抛出任何异常时，\_\_exit__()方法被执行。正如例子所示，异常抛出时，与之关联的type，value和stack trace传给__exit__()方法，因此抛出的ZeroDivisionError异常被打印出来了。开发库时，清理资源，关闭文件等等操作，都可以放在__exit__方法当中。
因此，Python的with语句是提供一个有效的机制，让代码更简练，同时在异常产生时，清理工作更简单。
