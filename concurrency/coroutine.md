# 5.1 协程

## 协程序概念
协程：
  - 定义：一个过程，与调用方协作，产出由调
用方提供的值
  - 语法：将 yield 用在表达式的右边
  - 状态：使用 inspect.getgeneratorstate() 确定
    - GEN_CREATED - 等待开始执行
    - GEN_RUNNING - 解释器正在执行
    - GEN_SUSPENDED - 在 yield 表达式处暂停
    - GEN_CLOSED - 执行结束

协程协议：
  - .next():
  - .send():
  - .throw():
  - .close():

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
