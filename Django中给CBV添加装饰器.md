### Django中给CBV添加装饰器

Django中给CBV添加装饰器，需要导入

```python
from django.utils.decorators import method_decorator

"""
CBV中django中不建议你直接给类添加装饰器
无论该装饰器能都正常给你，都不建议加
"""
# 在类上指定给哪个函数添加装饰器，
@method_decorator(login_auth,name='get') # 方式2（可以添加）
@method_decorator(other_decorator,name='post')
@method_decorator(login_auth,name='put')
class MyLogin(View):
  # 方式3:它会直接作用于当前类里面的所有的方法
  @method_decorator(login_auth)  
  def dispatch(request, *args, **kwargs):
    pass
  # 里面放的是装饰器
  @method_decorator(login_auth) # 方式1
  def get(self, request):
    pass
```

### 装饰器

​	装饰器的目标：就是在不修改被装饰对象源代码与调用方式的前提下为其添加新功能

```python
# 在被装饰对象正上方的单独一行写@装饰器名字
def timmer(func):
  def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
@timmer # 等价index = timmer(index)
def index(x,y,z):
  pass
# 
```

#### 当有多个装饰器的时候

```python
@deco1# deco1.wrapper返回的内存地址）index = deco1(index（deco2.wrapper返回的内存地址）)
@deco2  # （deco2.wrapper返回的内存地址）index = deco2(index（deco3.wrapper返回的内存地址）)
@deco3  # （deco3.wrapper返回的内存地址）index = deco3(index)从下往上依次
def index():
  pass

@classmethod
@permission
```

#### @print('hello')装饰对象时执行流程

```python
@print('hello')  # 先执行print('hello')函数，返回值x，然后执行@x
<==>@None  
```

### 无参