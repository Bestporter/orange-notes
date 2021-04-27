# `__slots__`

`__slots__`允许我们声明并限定类成员，并拒绝类创建`__dict__`和`__weakref__`属性以**节约内存空间**。

在python新式类中，可以定义一个变量`__slots__`，它的作用是阻止在实例化类时为实例分配dict，

默认情况下每个类都会有一个dict,通过`__dict__`访问，这个dict维护了这个实例的所有属性

**`__slots__`是一个元组，包括了当前能访问到的属性。**
当定义了slots后，slots中定义的变量变成了类的描述符，相当于java，c++中的成员变量声明，
类的实例只能拥有slots中定义的变量，不能再增加新的变量。注意：定义了slots后，就不再有dict。

如

```python
__slots__ = ('_items', '_call')
```

参考源码，operator中的itemgetter

```python
class itemgetter:
    """
    Return a callable object that fetches the given item(s) from its operand.
    After f = itemgetter(2), the call f(r) returns r[2].
    After g = itemgetter(2, 5, 3), the call g(r) returns (r[2], r[5], r[3])
    """
    __slots__ = ('_items', '_call')

    def __init__(self, item, *items):
        if not items:
            self._items = (item,)
            def func(obj):
                return obj[item]
            self._call = func
        else:
            self._items = items = (item,) + items
            def func(obj):
                return tuple(obj[i] for i in items)
            self._call = func

    def __call__(self, obj):
        return self._call(obj)

    def __repr__(self):
        return '%s.%s(%s)' % (self.__class__.__module__,
                              self.__class__.__name__,
                              ', '.join(map(repr, self._items)))

    def __reduce__(self):
        return self.__class__, self._items
```

