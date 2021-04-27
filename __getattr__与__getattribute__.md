## `__getattr__`

不存在的属性访问，触发getattr

```python
class Foo:
    def __init__(self, x):
        self.x = x

    def __getattr__(self, item):
        print('执行的是我')
        # return self.__dict__[item]


f1 = Foo(10)
```

## `getattribute`

查找属性无论是否存在，都会执行

```python
class Foo:
    def __init__(self, x):
        self.x = x

    def __getattr__(self, item):
        print('执行的是我')
        # return self.__dict__[item]

    def __getattribute__(self, item):
        print('不管是否存在，我都会执行')


f1 = Foo(10)
# 只执行了__getattribute__
print(f1.y)
不管是否存在，我都会执行
None
```

# 三、**getattr**与**getattribute**

- 当**getattribute**与**getattr**同时存在，只会执行**getattrbute**，除非**getattribute**在执行过程中抛出异常AttributeError

```python
class Foo:
    def __init__(self, x):
        self.x = x

    def __getattr__(self, item):
        print('执行的是我')
        # return self.__dict__[item]

    def __getattribute__(self, item):
        print('不管是否存在，我都会执行')
        raise AttributeError('error')


f1 = Foo(10)

print(f1.y)
不管是否存在，我都会执行
执行的是我
None
```

