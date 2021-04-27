判断字符串是否是以xxx开头的

```python
string = 'JIT'
match = ['JIT', 'FG', 'KKK']
any(map(string.startswith, match))

```



Q查询中

```python
q =  [    "78",    "95",    "77",    "91",    "92",    "93",    "94",    "75",    "27",    "28",    "45",    "89",    "10",    "51",    "02",    "60",    "27", ]

from re import escape as reescape 
result = Record14.objects.filter(code_postal__regex= '^({})'.format('|'.join(map(reescape, q))) )

# 对于给定的列表q，这将导致一个正则表达式：

'''
^(78|95|77|91|92|93|94|75|27|28|45|89|10|51|02|60|27)
^是此处的起始锚点，并且管道充当“联合”，因此此正则表达式查找以78，95，77等开头的列。
''''''
```

re的escape

```python
import re


print re.escape("Hello 123 .?!@ World")
'''
OUTPUT
Hello\ 123\ \.\?\!\@\ World
'''
```

`map(func, list)`,对list中的每一个元素都执行func方法

`reduce()`把结果继续和序列的下一个元素做累积计算，其效果就是：

js

```
[x1, x2, x3, x4].reduce(f) = f(f(f(x1, x2), x3), x4)
```

python 

reduce(func, list)

```python
#计算一个整数列表的乘积时。
from functools import reduce
product = reduce( (lambda x, y: x * y), [1, 2, 3, 4] )
```

filter

```python
number_list = range(-5, 5)
less_than_zero = filter(lambda x: x < 0, number_list)
print(list(less_than_zero))  
```



