# 5.1 协程
## 1. 协程序概念
### 1.1 协程的理解
协程理解：
  - 协程是用户空间的线程，编程语言必需提供接口实现线程调度
  - python 中 yield 是一种流程控制工具，使用它可以实现协程

### 1.2 协程状态
inspect.getgeneratorstate():
  - GEN_CREATED - 等待开始执行
  - GEN_RUNNING - 解释器正在执行
  - GEN_SUSPENDED - 在 yield 表达式处暂停
  - GEN_CLOSED - 执行结束


## 2. 协程协议：
### 2.1 预激协程
**.next()**:
  - 作用：预激(prime)协程
  - 效果：让协程向前执行到第一个 yield 表达式，准备好作为活跃的协程使用
  - == .send(None)

```python
>>> my_coro = simple_coroutine()
>>> my_coro.send(1729)
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: can not send non-None value to a just-started generator
```

**预激协程的装饰器**
```python
from functools import wraps

def coroutine(func):
    """装饰器：向前执行到第一个`yield`表达式，预激`func`"""
    @wraps(func)
    def primer(*args,**kwargs): ➊
        gen = func(*args,**kwargs) ➋
        next(gen) ➌
        return gen ➍
    return primer
```

**其他预激方式**
框架：
  - Tornado - tornado.gen 装饰器 <http://tornado.readthedocs.org/en/latest/gen.html>

yield from: 调用协程时，会自动预激

### 2.2 数据数据传递
**.send(value)**:
  - 作用：调用方向协程发送数据，value 成为生成器函数中 yield 表达式的值
  - 要求：仅当协程处于暂停状态时才能调用，即必需先调用 next()方法预激协程
  - eg: b = yield a
    - 协程向调用者返回 a，并在 yeild 处暂停
    - 调用 .send(value)后，value 值被赋给 b，协程向下运行到另一个 yeild 表达式

```python
>>> def simple_coro2(a):
       print('-> Started: a =', a)
       b = yield a
       print('-> Received: b =', b)
       c = yield a + b
       print('-> Received: c =', c)

>>> my_coro2 = simple_coro2(14)
>>> from inspect import getgeneratorstate
>>> getgeneratorstate(my_coro2)
'GEN_CREATED'
>>> next(my_coro2)  # 预激(prime)协程
  Started: a = 14  # 协程返回 a 的值 14
14
>>> getgeneratorstate(my_coro2)
'GEN_SUSPENDED'
>>> my_coro2.send(28) # b 被赋值 28，协程返回 a+b 的值
   Received: b = 28
42
```

### 2.3 协程终止
**终止方式**
  - 发送某个哨符值，让协程发生异常退出，内置的 None 和Ellipsis 等常量经常用作哨符值
  -

.close():
  - 作用：终止生成器

### 2.4 异常控制
**协程中的异常**：
  - 协程中未处理的异常会向上冒泡，传给 next 函数或 send 方法的调用方
  - 协程内没有处理异常，协程会终止; 重新激活协程，会抛出StopIteration异常

.throw(exc_type[, exc_value[, traceback]])
  - 作用：显式地把异常发给协程




python3 新特性：
  - 生成器可以返回一个值，以前，如果在生成器中给 return 语句提供值，会抛出SyntaxError 异常
  - 引入了 yield from 句法，可以把复杂的生成器重构成小型的嵌套生成器，省去了之前
  把 *生成器的工作委托给子生成器* 所需的大量样板代码

## 协程协议

## yield from
### 作用：
1.  yield from 结构会在内部自动捕获 StopIteration 异常。这种处理方式与
for 循环处理 StopIteration 异常的方式一样循环机制使用用户易于理解的方式处理异
常。对 yield from 结构来说，解释器不仅会捕获 StopIteration 异常，还会把 value 属性
的值变成 yield from 表达式的值
2. yield from 的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来，
这样二者可以直接发送和产出值，还可以直接传入异常，而不用在位于中间的协程中添加
大量处理异常的样板代码

### 意义：
1. 子生成器产出的值都直接传给委派生成器的调用方（即客户端代码）
2. 使用 send() 方法发给委派生成器的值都直接传给子生成器。如果发送的值是 None，那
么会调用子生成器的 __next__() 方法。如果发送的值不是 None，那么会调用子生成器
的 send() 方法。如果调用的方法抛出 StopIteration 异常，那么委派生成器恢复运行。
任何其他异常都会向上冒泡，传给委派生成器
3. 生成器退出时，生成器（或子生成器）中的 return expr 表达式会触发 StopIteration(expr)
4. yield from 表达式的值是子生成器终止时传给 StopIteration 异常的第一个参数
5. 传入委派生成器的异常，除了 GeneratorExit 之外都传给子生成器的 throw() 方法如
果调用 throw() 方法时抛出 StopIteration 异常，委派生成器恢复运行。 StopIteration
之外的异常会向上冒泡，传给委派生成器
异常抛出
6. 如果把 GeneratorExit 异常传入委派生成器，或者在委派生成器上调用 close() 方法，
那么在子生成器上调用 close() 方法，如果它有的话。如果调用 close() 方法导致异常
抛出，那么异常会向上冒泡，传给委派生成器；否则，委派生成器抛出 GeneratorExit
异常


## 附注
协程：
  - <https://www.python.org/dev/peps/pep-0380/>
