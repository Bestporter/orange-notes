# yield from 

`yield from` 是在Python3.3才出现的语法。所以这个特性在Python2中是没有的。

`yield from` 后面需要加的是可迭代对象，它可以是普通的可迭代对象，也可以是迭代器，甚至是生成器。

使用`yield`

```python
# 字符串
astr='ABC'
# 列表
alist=[1,2,3]
# 字典
adict={"name":"wangbm","age":18}
# 生成器
agen=(i for i in range(4,8))

def gen(*args, **kw):
    for item in args:
        for i in item:
            yield i

new_list=gen(astr, alist, adict， agen)
print(list(new_list))
# ['A', 'B', 'C', 1, 2, 3, 'name', 'age', 4, 5, 6, 7]
复制代码
```

使用`yield from`

```python
# 字符串
astr='ABC'
# 列表
alist=[1,2,3]
# 字典
adict={"name":"wangbm","age":18}
# 生成器
agen=(i for i in range(4,8))

def gen(*args, **kw):
    for item in args:
        yield from item

new_list=gen(astr, alist, adict, agen)
print(list(new_list))
# ['A', 'B', 'C', 1, 2, 3, 'name', 'age', 4, 5, 6, 7]
复制代码
```

由上面两种方式对比，可以看出，**yield from后面加上可迭代对象，他可以把可迭代对象里的每个元素一个一个的yield出来**，对比yield来说代码更加简洁，结构更加清晰。

> 作者：王一白
> 链接：https://juejin.cn/post/6844903632534503437
> 来源：掘金
> 著作权归作者所有。



### 生成器的嵌套

当 `yield from` 后面加上一个生成器后，就实现了生成的嵌套。

当然实现生成器的嵌套，并不是一定必须要使用`yield from`，而是使用`yield from`可以让我们避免让我们自己处理各种料想不到的异常，而让我们专注于业务代码的实现。

> 1、`调用方`：调用委派生成器的客户端（调用方）代码
> 2、`委托生成器`：包含yield from表达式的生成器函数
> 3、`子生成器`：yield from后面加的生成器函数

这个例子，是实现实时计算平均值的。
比如，第一次传入10，那返回平均数自然是10.
第二次传入20，那返回平均数是(10+20)/2=15
第三次传入30，那返回平均数(10+20+30)/3=20

```python
# 子生成器
def average_gen():
    total = 0
    count = 0
    average = 0
    while True:
        new_num = yield average
        count += 1
        total += new_num
        average = total/count

# 委托生成器
def proxy_gen():
    while True:
        yield from average_gen()

# 调用方
def main():
    calc_average = proxy_gen()
    next(calc_average)            # 预激下生成器
    print(calc_average.send(10))  # 打印：10.0
    print(calc_average.send(20))  # 打印：15.0
    print(calc_average.send(30))  # 打印：20.0

if __name__ == '__main__':
    main()
```

认真阅读以上代码，你应该很容易能理解，调用方、委托生成器、子生成器之间的关系。我就不多说了

**委托生成器的作用是**：在调用方与子生成器之间建立一个`双向通道`。

#### 所谓的双向通道是什么意思呢？
​		**调用方可以通过`send()`直接发送消息给子生成器，而子生成器yield的值，也是直接返回给调用方。**

你可能会经常看到有些代码，还可以在`yield from`前面看到可以赋值。这是什么用法？

**你可能会以为，子生成器yield回来的值，被委托生成器给拦截了。你可以亲自写个demo运行试验一下，并不是你想的那样。**
因为我们之前说了，**委托生成器，只起一个桥梁作用**，它建立的是一个`双向通道`，它并没有权利也没有办法，对子生成器yield回来的内容做拦截。

```python
# 子生成器
def average_gen():
    total = 0
    count = 0
    average = 0
    while True:
        new_num = yield average
        if new_num is None:
            break
        count += 1
        total += new_num
        average = total/count

    # 每一次return，都意味着当前协程结束。
    return total,count,average

# 委托生成器
def proxy_gen():
    while True:
        # 只有子生成器要结束（return）了，yield from左边的变量才会被赋值，后面的代码才会执行。
        total, count, average = yield from average_gen()
        print("计算完毕！！\n总共传入 {} 个数值， 总和：{}，平均数：{}".format(count, total, average))

# 调用方
def main():
    calc_average = proxy_gen()
    next(calc_average)            # 预激协程
    print(calc_average.send(10))  # 打印：10.0
    print(calc_average.send(20))  # 打印：15.0
    print(calc_average.send(30))  # 打印：20.0
    calc_average.send(None)      # 结束协程
    # 如果此处再调用calc_average.send(10)，由于上一协程已经结束，将重开一协程

if __name__ == '__main__':
    main()
    
    
10.0
15.0
20.0
计算完毕！！
总共传入 3 个数值， 总和：60，平均数：20.0
```

**yield from最重要的作用就是提供了一个“数据传输的管道”**



## 为什么需要next（）或者send（None）来启动生成器？

第一次调用时，请使用next()语句或是send(None)，不能使用send发送一个非None的值，否则会出错的，因为没有Python yield语句来接收这个值。

下面说明下send执行的顺序。先记住，**n1 = yield r这句话是从右往左执行的**。当第一次send（None）（对应11行）时，启动生成器，从生成器函数的第一行代码开始执行，直到第一次执行完yield（对应第4行）后，跳出生成器函数。这个过程中，n1一直没有定义。

运行到send（1）时，进入生成器函数，此时，**将yield r看做一个整体，赋值给它并且传回**。此时即相当于把1赋值给n1，但是并不执行yield部分。下面继续从yield的下一语句继续执行，然后重新运行到yield语句，执行后，跳出生成器函数。即send和next相比，只是开始多了一次赋值的动作，其他运行流程是相同的。

```python
def consumer():
     r = 'here'
     while True:
        n1 = yield r   #这里的等式右边相当于一个整体，接受回传值  line4
         if not n1:
             return
         print('[CONSUMER] Consuming %s...' % n1)
         r = '%d00 OK' % n1

def produce(c):
    aa = c.send(None)# line 11
    n = 0
    while n < 5:
       n = n + 1
       print('[PRODUCER] Producing %s...' % n)
       r1 = c.send(n)
       print('[PRODUCER] Consumer return: %s' % r1)
    c.close()
c = consumer()
produce(c)
```

## 为什么要使用yield from

学到这里，我相信你肯定要问，既然委托生成器，起到的只是一个双向通道的作用，我还需要委托生成器做什么？我调用方直接调用子生成器不就好啦？

高能预警~~~

下面我们来一起探讨一下，到底yield from 有什么过人之处，让我们非要用它不可。

### . 因为它可以帮我们处理异常

如果我们去掉委托生成器，而直接调用子生成器。那我们就需要把代码改成像下面这样，我们需要自己捕获异常并处理。而不像使`yield from`那样省心。

```python
# 子生成器
def average_gen():
    total = 0
    count = 0
    average = 0
    while True:
        new_num = yield average
        if new_num is None:
            break
        count += 1
        total += new_num
        average = total/count
    return total,count,average

# 调用方
def main():
    calc_average = average_gen()
    next(calc_average)            # 预激协程
    print(calc_average.send(10))  # 打印：10.0
    print(calc_average.send(20))  # 打印：15.0
    print(calc_average.send(30))  # 打印：20.0

    # ----------------注意-----------------
    try:
        calc_average.send(None)
    except StopIteration as e:
        total, count, average = e.value
        print("计算完毕！！\n总共传入 {} 个数值， 总和：{}，平均数：{}".format(count, total, average))
    # ----------------注意-----------------

if __name__ == '__main__':
    main()
```




此时的你，可能会说，不就一个`StopIteration`的异常吗？自己捕获也没什么大不了的。

你要是知道`yield from`在背后为我们默默无闻地做了哪些事，你就不会这样说了。

### 具体`yield from`为我们做了哪些事(源码)

可以参考如下这段代码。

```python
#一些说明
"""
_i：子生成器，同时也是一个迭代器
_y：子生成器生产的值
_r：yield from 表达式最终的值
_s：调用方通过send()发送的值
_e：异常对象
"""

_i = iter(EXPR)

try:
    _y = next(_i)
except StopIteration as _e:
    _r = _e.value

else:
    while 1:
        try:
            _s = yield _y
        except GeneratorExit as _e:
            try:
                _m = _i.close
            except AttributeError:
                pass
            else:
                _m()
            raise _e
        except BaseException as _e:
            _x = sys.exc_info()
            try:
                _m = _i.throw
            except AttributeError:
                raise _e
            else:
                try:
                    _y = _m(*_x)
                except StopIteration as _e:
                    _r = _e.value
                    break
        else:
            try:
                if _s is None:
                    _y = next(_i)
                else:
                    _y = _i.send(_s)
            except StopIteration as _e:
                _r = _e.value
                break
RESULT = _r
```

