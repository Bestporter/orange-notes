# Python cookbook 笔记

## 第一章

```python
lst = [1,2,3]
a,b,c = lst
from collections import deque
#写查询元素的代码时，通常会使用包含 yield 表达式的生成器函数
def search(lines, pattern, history=5):
  #使用 deque(maxlen=N) 构造函数会新建一个固定大小的队列。当新的元素加入并且这个队列已满的时候， 最老的元素会自动被移除掉。
  que = deque(maxlen=history)
  for i in lines:
    if patter in i:
        yield i, que
    que.append(i)

# 怎样从一个集合中获得最大或者最小的 N 个元素列表？
import heapq
# heapq 模块有两个函数：nlargest() 和 nsmallest() 可以完美解决这个问题。
portfolio = [
    {'name': 'IBM', 'shares': 100, 'price': 91.1},
    {'name': 'AAPL', 'shares': 50, 'price': 543.22},
    {'name': 'FB', 'shares': 200, 'price': 21.09},
    {'name': 'HPQ', 'shares': 35, 'price': 31.75},
    {'name': 'YHOO', 'shares': 45, 'price': 16.35},
    {'name': 'ACME', 'shares': 75, 'price': 115.65}
]
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])
```

heapq.heappush() 和heapq.heappop()分別在隊列_queue上插入和刪除第一個元素，

### 实现一个优先级队列

队列包含了一个 `(-priority, index, item)` 的元组。 优先级为负数的目的是使得元素按照优先级从高到低排序。 这个跟普通的按优先级从低到高排序的堆排序恰巧相反。

```python
import heapq

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0

    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item))
        self._index += 1

    def pop(self):
        return heapq.heappop(self._queue)[-1]
      
>>> class Item:
...     def __init__(self, name):
...         self.name = name
...     def __repr__(self):
...         return 'Item({!r})'.format(self.name)
...
>>> q = PriorityQueue()
>>> q.push(Item('foo'), 1)
>>> q.push(Item('bar'), 5)
>>> q.push(Item('spam'), 4)
>>> q.push(Item('grok'), 1)
>>> q.pop()
Item('bar')
>>> q.pop()
Item('spam')
>>> q.pop()
Item('foo')
>>> q.pop()
Item('grok')
>>>
```

`index` 变量的作用是保证同等优先级元素的正确排序。 通过保存一个不断增加的 `index` 下标变量，可以确保元素按照它们插入的顺序排序。 而且， `index` 变量也在相同优先级元素比较的时候起到重要作用。

如果你使用元组 `(priority, item)` ，只要两个元素的优先级不同就能比较。 但是如果两个元素优先级一样的话，那么比较操作就会跟之前一样出错：

通过引入另外的 `index` 变量组成三元组 `(priority, index, item)` ，就能很好的避免上面的错误， 因为不可能有两个元素有相同的 `index` 值。Python 在做元组比较时候，如果前面的比较已经可以确定结果了， 后面的比较操作就不会发生了,就能完美避免item无法进行比较的问题。

### 怎样实现一个键对应多个值的字典（也叫 `multidict`）？

​	`collections`中的`defaultdict` 会自动为将要访问的键（就算目前字典中并不存在这样的键）创建映射实体。 

​	如果你并不需要这样的特性，你可以在一个普通的字典上使用 `setdefault()` 方法来代替。

```python
from collections import defaultdict

d = defaultdict(list)
d['a'].append(1)
d['a'].append(2)
d['b'].append(4)

d = defaultdict(set)
d['a'].add(1)
d['a'].add(2)
d['b'].add(4)

d = {} # 一个普通的字典
d.setdefault('a', []).append(1)
d.setdefault('a', []).append(2)
d.setdefault('b', []).append(4)
```

​	创建一个多值映射字典是很简单的。但是，如果你选择自己实现的话，那么对于值的初始化可能会有点麻烦， 你可能会像下面这样来实现：

```python
d = {}
for key, value in pairs:
    if key not in d:
        d[key] = []
    d[key].append(value)
```

如果使用 `defaultdict` 的话代码就更加简洁了：

```python
d = defaultdict(list)
for key, value in pairs:
    d[key].append(value)
```

### 你想创建一个字典，并且在迭代或序列化(如json.dumps)这个字典的时候能够控制元素的顺序。

为了能控制一个字典中元素的顺序，你可以使用 `collections` 模块中的 `OrderedDict` 类。 在迭代操作的时候它会保持元素被插入时的顺序，示例如下：

```python
from collections import OrderedDict

d = OrderedDict()
d['foo'] = 1
d['bar'] = 2
d['spam'] = 3
d['grok'] = 4
# Outputs "foo 1", "bar 2", "spam 3", "grok 4"
for key in d:
    print(key, d[key])
```

当你想要构建一个将来需要序列化或编码成其他格式的映射的时候， `OrderedDict` 是非常有用的。 比如，你想精确控制以 JSON 编码后字段的顺序，你可以先使用 `OrderedDict` 来构建这样的数据：

```python
>>> import json
>>> json.dumps(d)
'{"foo": 1, "bar": 2, "spam": 3, "grok": 4}'
>>>
```

`OrderedDict` 内部维护着一个根据键插入顺序排序的双向链表。每次当一个新的元素插入进来的时候， 它会被放到链表的尾部。对于一个已经存在的键的重复赋值不会改变键的顺序。

需要注意的是，**一个 `OrderedDict` 的大小是一个普通字典的两倍**，因为它**内部维护着另外一个链表**。 所以如果你要构建一个需要大量 `OrderedDict` 实例的数据结构的时候（比如读取 100,000 行 CSV 数据到一个 `OrderedDict` 列表中去）， 那么你就得仔细权衡一下是否使用 `OrderedDict` 带来的好处要大过额外内存消耗的影响。

### 怎样在数据字典中执行一些计算操作（比如求最小值、最大值、排序等等）？

为了对字典值执行计算操作，通常需要使用 `zip()` 函数先将键和值反转过来。 比如，下面是查找最小和最大股票价格和股票值的代码：

类似的，可以使用 `zip()` 和 `sorted()` 函数来排列字典数据：

```python
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}
# 利用元组比较的时候，先按顺序比较的特性。
min_price = min(zip(prices.values(), prices.keys()))
# min_price is (10.75, 'FB')
max_price = max(zip(prices.values(), prices.keys()))
# max_price is (612.78, 'AAPL')
prices_sorted = sorted(zip(prices.values(), prices.keys()))
# prices_sorted is [(10.75, 'FB'), (37.2, 'HPQ'),
#                   (45.23, 'ACME'), (205.55, 'IBM'),
#                   (612.78, 'AAPL')]
```

执行这些计算的时候，需要**注意的是 `zip()` 函数创建的是一个只能访问一次的迭代器**。 比如，下面的代码就会产生错误：

```python
prices_and_names = zip(prices.values(), prices.keys())
print(min(prices_and_names)) # OK
print(max(prices_and_names)) # ValueError: max() arg is an empty sequence
```

你可以在 `min()` 和 `max()` 函数中提供 `key` 函数参数来获取最小值或最大值对应的键的信息。比如：

```python
min(prices, key=lambda k: prices[k]) # Returns 'FB'
max(prices, key=lambda k: prices[k]) # Returns 'AAPL'
```

但是，如果还想要得到最小值，你又得执行一次查找操作。比如：

```python
min_value = prices[min(prices, key=lambda k: prices[k])]
```

需要注意的是在计算操作中使用到了 (值，键) 对。当多个实体拥有相同的值的时候，键会决定返回结果。 比如，在执行 `min()` 和 `max()` 操作的时候，如果恰巧最小或最大值有重复的，那么拥有最小或最大键的实体会返回：

```python
>>> prices = { 'AAA' : 45.23, 'ZZZ': 45.23 }
>>> min(zip(prices.values(), prices.keys()))
(45.23, 'AAA')
>>> max(zip(prices.values(), prices.keys()))
(45.23, 'ZZZ')
>>>
```

### 怎样在两个字典中寻寻找相同点（比如相同的键、相同的值等等）？

```python
a = {
    'x' : 1,
    'y' : 2,
    'z' : 3
}

b = {
    'w' : 10,
    'x' : 11,
    'y' : 2
}
# 以下返回的均为set数据结构
# Find keys in common
b = a.keys() & b.keys() # { 'x', 'y' }
print(type(b))  # <class 'set'>
# Find keys in a that are not in b
a.keys() - b.keys() # { 'z' }
# Find (key,value) pairs in common 
a.items() & b.items() # { ('y', 2) }   set(tuple)
```

这些操作也可以用于修改或者过滤字典元素。 比如，假如你想以现有字典构造一个排除几个指定键的新字典。 下面利用字典推导来实现这样的需求：

```python
# Make a new dictionary with certain keys removed
c = {key:a[key] for key in a.keys() - {'z', 'w'}}
# c is {'x': 1, 'y': 2}
```

一个字典就是一个键集合与值集合的映射关系。 字典的 `keys()` 方法返回一个展现键集合的键视图对象。 键视图的一个很少被了解的特性就是它们也支持集合操作，比如集合并、交、差运算。 所以，如果你想对集合的键执行一些普通的集合操作，可以直接使用键视图对象而不用先将它们转换成一个 set。

字典的 `items()` 方法返回一个包含 (键，值) 对的元素视图对象。 这个对象同样也支持集合操作，并且可以被用来查找两个字典有哪些相同的键值对。

**尽管字典的 `values()` 方法也是类似，但是它并不支持这里介绍的集合操作。** 某种程度上是因为值视图不能保证所有的值互不相同，这样会导致某些集合操作会出现问题。 不过，**如果你硬要在值上面执行这些集合操作的话，你可以先将值集合转换成 set，然后再执行集合运算就行了。**

### 怎样在一个序列上面保持元素顺序的同时消除重复的值？

如果序列上的值都是 `hashable` 类型，那么可以很简单的利用集合或者生成器来解决这个问题。比如：

```python
def dedupe(items):
    seen = set()
    for item in items:
        if item not in seen:
            yield item
            seen.add(item)
>>> a = [1, 5, 2, 1, 9, 1, 5, 10]
>>> list(dedupe(a))
[1, 5, 2, 9, 10]
>>>
```

这个方法仅仅在序列中元素为 `hashable` 的时候才管用。 如果你想消除元素不可哈希（比如 `dict` 类型）的序列中重复元素的话，你需要将上述代码稍微改变一下，就像这样：

```python
# key为一个函数
def dedupe(items, key=None):
    seen = set()
    for item in items:
      	# 使用key函数按其规则，将item中的元素组成元组（hashable的数据结构）添加到set中，用来判重
        val = item if key is None else key(item)
        if val not in seen:
            yield item
            seen.add(val)
a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
list(dedupe(a, key=lambda d: (d['x'],d['y'])))
list(dedupe(a, key=lambda d: d['x']))
```

如果你想基于单个字段、属性或者某个更大的数据结构来消除重复元素，第二种方案同样可以胜任。

如果你仅仅就是想消除重复元素，通常可以简单的构造一个集合。比如：

```python
>>> a
[1, 5, 2, 1, 9, 1, 5, 10]
>>> set(a)
{1, 2, 10, 5, 9}
>>>
```

然而，这种方法不能维护元素的顺序，生成的结果中的元素位置被打乱。而上面的方法可以避免这种情况。

在本节中我们使用了生成器函数让我们的函数更加通用，不仅仅是局限于列表处理。 比如，如果如果你想读取一个文件，消除重复行，你可以很容易像这样做：

```python
with open(somefile,'r') as f:
	for line in dedupe(f):
  	...
```

上述key函数参数模仿了 `sorted()` , `min()` 和 `max()` 等内置函数的相似功能。 可以参考 1.8 和 1.13 小节了解更多。



### 如果你的程序包含了大量无法直视的硬编码切片，并且你想清理一下代码。

**slice()** 函数实现切片对象，主要用在切片操作函数里的参数传递。

```python
class slice(stop)
class slice(start, stop[, step])
#返回一个切片对象。
```

```python
>>>myslice = slice(5)    # 设置截取5个元素的切片
>>> myslice
slice(None, 5, None)
>>> arr = range(10)
>>> arr
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> arr[myslice]         # 截取 5 个元素
[0, 1, 2, 3, 4]
>>>
```

假定你要从一个记录（比如文件或其他类似格式）中的某些固定位置提取字段：

```python
######    0123456789012345678901234567890123456789012345678901234567890'
record = '....................100 .......513.25 ..........'
cost = int(record[20:23]) * float(record[31:37])
# 与其那样写，为什么不像这样命名切片呢：
SHARES = slice(20, 23)
PRICE = slice(31, 37)
cost = int(record[SHARES]) * float(record[PRICE])
# 在这个版本中，你避免了使用大量难以理解的硬编码下标。这使得你的代码更加清晰可读。
```

​	一般来讲，代码中如果出现大量的**硬编码下标**会使得代码的可读性和可维护性大大降低。 比如，如果你回过来看看一年前你写的代码，你会摸着脑袋想那时候自己到底想干嘛啊。 这是一个很简单的解决方案，它让你更加清晰的表达代码的目的。

内置的 `slice()` 函数创建了一个切片对象。所有使用切片的地方都可以使用切片对象。比如：

```python
>>> items = [0, 1, 2, 3, 4, 5, 6]
>>> a = slice(2, 4)
>>> items[2:4]
[2, 3]
>>> items[a]
[2, 3]
>>> items[a] = [10,11]
>>> items
[0, 1, 10, 11, 4, 5, 6]
>>> del items[a]
>>> items
[0, 1, 4, 5, 6]
```

如果你有一个切片对象a，你可以分别调用它的 `a.start` , `a.stop` , `a.step` 属性来获取更多的信息。比如：

```python
>>> a = slice(5, 50, 2)
>>> a.start
5
>>> a.stop
50
>>> a.step
2
>>>
```

另外，你还可以通过调用切片的 `indices(size)` 方法将它映射到一个已知大小的序列上。 这个方法返回一个三元组 `(start, stop, step)` ，所有的值都会被缩小，直到适合这个已知序列的边界为止。 这样，使用的时就不会出现 `IndexError` 异常。比如：

```python
>>> a = slice(5, 50, 2)
>>> s = 'HelloWorld'
>>> a.indices(len(s))
(5, 10, 2)
>>> for i in range(*a.indices(len(s))):
...     print(s[i])
...
W
r
d
```

### 怎样找出一个序列中出现次数最多的元素呢？

**`collections.Counter` 类**就是专门为这类问题而设计的， 它甚至有一个有用的 **`most_common()` 方法直接给了你答案。**

为了演示，先假设你有一个单词列表并且想找出哪个单词出现频率最高。你可以这样做：

```python
words = [
    'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
    'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
    'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
    'my', 'eyes', "you're", 'under'
]
from collections import Counter
word_counts = Counter(words)
# 出现频率最高的3个单词
top_three = word_counts.most_common(3)
print(top_three)
# Outputs [('eyes', 8), ('the', 5), ('look', 4)]
```

作为输入， `Counter` 对象可以接受任意的由可哈希（`hashable`）元素构成的序列对象。 在底层实现上，**一个 `Counter` 对象就是一个字典**，将元素映射到它出现的次数上。比如：

```python
>>> word_counts['not']
1
>>> word_counts['eyes']
8
>>>
```

如果你想手动增加计数，可以简单的用加法：

```python
>>> morewords = ['why','are','you','not','looking','in','my','eyes']
>>> for word in morewords:
...     word_counts[word] += 1
...
>>> word_counts['eyes']
9
>>>
```

或者你可以使用 `update()` 方法：

```python
>>> word_counts.update(morewords) # 将morewords也添加进计数中
>>>
```

`Counter` 实例一个鲜为人知的特性是它们可以很容易的跟数学运算操作相结合。比如：

```python
>>> a = Counter(words)
>>> b = Counter(morewords)
>>> a
Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2,
"you're": 1, "don't": 1, 'under': 1, 'not': 1})
>>> b
Counter({'eyes': 1, 'looking': 1, 'are': 1, 'in': 1, 'not': 1, 'you': 1,
'my': 1, 'why': 1})
>>> # Combine counts
>>> c = a + b
>>> c
Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2,
'around': 2, "you're": 1, "don't": 1, 'in': 1, 'why': 1,
'looking': 1, 'are': 1, 'under': 1, 'you': 1})
>>> # Subtract counts
>>> d = a - b
>>> d
Counter({'eyes': 7, 'the': 5, 'look': 4, 'into': 3, 'my': 2, 'around': 2,
"you're": 1, "don't": 1, 'under': 1})
>>>
```

毫无疑问， `Counter` 对象在几乎所有需要制表或者计数数据的场合是非常有用的工具。 在解决这类问题的时候你应该优先选择它，而不是手动的利用字典去实现。

### 你有一个字典列表，你想根据某个或某几个字典字段来排序这个列表

通过使用 `operator` 模块的 `itemgetter` 函数，可以非常容易的排序这样的数据结构。 假设你从数据库中检索出来网站会员信息列表，并且以下列的数据结构返回：

```python
rows = [
    {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
    {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]
from operator import itemgetter
rows_by_fname = sorted(rows, key=itemgetter('fname'))
rows_by_uid = sorted(rows, key=itemgetter('uid'))
print(rows_by_fname)
print(rows_by_uid)
[{'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
{'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
{'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
{'fname': 'John', 'uid': 1001, 'lname': 'Cleese'}]

[{'fname': 'John', 'uid': 1001, 'lname': 'Cleese'},
{'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
{'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
{'fname': 'Big', 'uid': 1004, 'lname': 'Jones'}]
# itemgetter() 函数也支持多个 keys，比如下面的代码
rows_by_lfname = sorted(rows, key=itemgetter('lname', 'fname'))
print(rows_by_lfname)
[{'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
{'fname': 'John', 'uid': 1001, 'lname': 'Cleese'},
{'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
{'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'}]
```

在上面例子中， `rows` 被传递给接受一个关键字参数的 `sorted()` 内置函数。 这个参数是 `callable` 类型，并且从 `rows` 中接受一个单一元素，然后返回被用来排序的值。 `itemgetter()` 函数就是负责创建这个 `callable` 对象的。

`operator.itemgetter()` 函数有一个被 `rows` 中的记录用来查找值的索引参数。可以是一个字典键名称， 一个整形值或者任何能够传入一个对象的 `__getitem__()` 方法的值。 如果你传入多个索引参数给 `itemgetter()` ，它生成的 `callable` 对象会返回一个包含所有元素值的元组， 并且 `sorted()` 函数会根据这个元组中元素顺序去排序。 但你想要同时在几个字段上面进行排序（比如通过姓和名来排序，也就是例子中的那样）的时候这种方法是很有用的。

`itemgetter()` 有时候也可以用 `lambda` 表达式代替，比如：

```python
rows_by_fname = sorted(rows, key=lambda r: r['fname'])
rows_by_lfname = sorted(rows, key=lambda r: (r['lname'],r['fname']))
```

这种方案也不错。但是，**使用 `itemgetter()` 方式会运行的稍微快点**。因此，如果你对性能要求比较高的话就使用 `itemgetter()` 方式。

最后，**不要忘了这节中展示的技术也同样适用于 `min()` 和 `max()` 等函数**。比如：

```python
>>> min(rows, key=itemgetter('uid'))
{'fname': 'John', 'lname': 'Cleese', 'uid': 1001}
>>> max(rows, key=itemgetter('uid'))
{'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
```

### 你想排序类型相同的对象，但是他们不支持原生的比较操作。

内置的 `sorted()` 函数有一个关键字参数 `key` ，可以传入一个 `callable` 对象给它， **这个 `callable` 对象对每个传入的对象返回一个值**，**这个值会被 `sorted` 用来排序这些对象**。 比如，如果你在应用程序里面有一个 `User` 实例序列，并且你希望通过他们的 `user_id` 属性进行排序， 你可以提供一个以 `User` 实例作为输入并输出对应 `user_id` 值的 `callable` 对象。

```python
class User:
    def __init__(self, user_id):
        self.user_id = user_id

    def __repr__(self):
        return 'User({})'.format(self.user_id)


def sort_notcompare():
    users = [User(23), User(3), User(99)]
    print(users)
    print(sorted(users, key=lambda u: u.user_id))
```

另外一种方式是使用 `operator.attrgetter()` 来代替 lambda 函数：

```python
>>> from operator import attrgetter
>>> sorted(users, key=attrgetter('user_id'))
[User(3), User(23), User(99)]
>>>
```

选择使用 lambda 函数或者是 `attrgetter()` 可能取决于个人喜好。 但是， `attrgetter()` 函数通常会运行的快点，并且还能同时允许多个字段进行比较。 这个跟 `operator.itemgetter()` 函数作用于字典类型很类似（参考1.13小节）。 例如，如果 `User` 实例还有一个 `first_name` 和 `last_name` 属性，那么可以向下面这样排序：

```python
by_name = sorted(users, key=attrgetter('last_name', 'first_name'))
```

同样需要注意的是，这一小节用到的技术同样适用于像 `min()` 和 `max()` 之类的函数。比如：

```python
>>> min(users, key=attrgetter('user_id'))
User(3)
>>> max(users, key=attrgetter('user_id'))
User(99)
>>>
```

### 你有一个字典或者实例的序列，然后你想根据某个特定的字段比如 `date` 来分组迭代访问。

`itertools.groupby()` 函数对于这样的数据分组操作非常实用。 为了演示，假设你已经有了下列的字典列表：

现在假设你想在按 date 分组后的数据块上进行迭代。为了这样做，**你首先需要按照指定的字段(这里就是 `date` )排序， 然后调用 `itertools.groupby()` 函数：**

```python
rows = [
    {'address': '5412 N CLARK', 'date': '07/01/2012'},
    {'address': '5148 N CLARK', 'date': '07/04/2012'},
    {'address': '5800 E 58TH', 'date': '07/02/2012'},
    {'address': '2122 N CLARK', 'date': '07/03/2012'},
    {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
    {'address': '1060 W ADDISON', 'date': '07/02/2012'},
    {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
    {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
]
from operator import itemgetter
from itertools import groupby

# Sort by the desired field first
rows.sort(key=itemgetter('date'))
# Iterate in groups
for date, items in groupby(rows, key=itemgetter('date')):
    print(date)
    for i in items:
        print(' ', i)
"""
07/01/2012
  {'address': '5412 N CLARK', 'date': '07/01/2012'}
  {'address': '4801 N BROADWAY', 'date': '07/01/2012'}
07/02/2012
  {'address': '5800 E 58TH', 'date': '07/02/2012'}
  {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'}
  {'address': '1060 W ADDISON', 'date': '07/02/2012'}
07/03/2012
  {'address': '2122 N CLARK', 'date': '07/03/2012'}
07/04/2012
  {'address': '5148 N CLARK', 'date': '07/04/2012'}
  {'address': '1039 W GRANVILLE', 'date': '07/04/2012'}
"""
```

`groupby()` 函数扫描整个序列并且查找连续相同值（或者根据指定 key 函数返回值相同）的元素序列。 在每次迭代的时候，它会返回一个值和一个迭代器对象， 这个迭代器对象可以生成元素值全部等于上面那个值的组中所有对象。

一个非常重要的准备步骤是要根据指定的字段将数据排序。 因为 `groupby()` 仅仅检查连续的元素，如果事先并没有排序完成的话，分组函数将得不到想要的结果。

如果你仅仅只是想根据 `date` 字段将数据分组到一个大的数据结构中去，并且允许随机访问， 那么你最好使用 `defaultdict()` 来构建一个多值字典，关于多值字典已经在 1.6 小节有过详细的介绍。比如：

```
from collections import defaultdict
rows_by_date = defaultdict(list)
for row in rows:
    rows_by_date[row['date']].append(row)
```

这样的话你可以很轻松的就能对每个指定日期访问对应的记录：

```
>>> for r in rows_by_date['07/01/2012']:
... print(r)
...
{'date': '07/01/2012', 'address': '5412 N CLARK'}
{'date': '07/01/2012', 'address': '4801 N BROADWAY'}
>>>
```

在上面这个例子中，我们没有必要先将记录排序。因此，如果对内存占用不是很关心， 这种方式会比先排序然后再通过 `groupby()` 函数迭代的方式运行得快一些。