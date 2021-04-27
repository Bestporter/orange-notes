# getattr与setattr

f1.attribute = 10，执行的时候调用的setattr方法，传入的key为‘attribute’字符串，value是正常的数值

```python
class Fo:
    x = 1

    def __init__(self, y):
        self.y = y

    def __getattr__(self, item):
        print('----> from getattr:你找的属性不存在')

    def __setattr__(self, key, value):
        print('----> from setattr')
        # self.key = value  # 这就无限递归了,你好好想想
        self.__dict__[key] = value  # 应该使用它

    def __delattr__(self, item):
        print('----> from delattr')
        # del self.item  # 无限递归了
        self.__dict__.pop(item)
f1 = Fo(10)
print(f1.__dict__)
f1.attribute = 10
print(f1.__dict__)
----> from setattr
{'y': 10}
----> from setattr
{'y': 10, 'attribute': 10}
```

## setattr

添加/修改属性的时候会触发它的执行

**重写此方法之后，需要操作属性字典进行赋值，否则永远无法赋值**

```python
# 因为你重写了__setattr__，凡是赋值操作都会触发它的运行，你啥都没写，就是根本没赋值，除非你直接操作属性字典，否则永远无法赋值
print(f1.__dict__)  
f1.z = 3
print(f1.__dict__)
```

## delattr

删除属性

```python 
del f1.y
```

## getattr

只有在使用点属性，且属性不存在的时候才会出发，一般属性存在的情况下都是触发getattribute函数调用属性，当发现属性不存在，才会触发getattr