# delitem 与delattr

Deleted,setitem,getitem,这些方法都是在与中括号取值的时候进行调用的。

```python
class Foo:
    def __init__(self, name):
        self.name = name

    def __getitem__(self, item):
        print('getitem执行', self.__dict__[item])

    def __setitem__(self, key, value):
        print('setitem执行')
        self.__dict__[key] = value

    def __delitem__(self, key):
        print('del obj[key]时，delitem执行')
        self.__dict__.pop(key)

    def __delattr__(self, item):
        print('del obj.key时，delattr执行')
        self.__dict__.pop(item)


f1 = Foo('sb')
```