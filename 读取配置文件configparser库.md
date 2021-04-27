# 读取配置文件>configparser库

问题：怎样读取普通.ini格式的配置文件？

```python
import os
import configparser


class Config(object):
    def __init__(self, config_file='config.ini'):
      	# 初始化路径
        self._path = os.path.join(os.getcwd(), config_file)
        
        # 判断是否存在文件
        if not os.path.exists(self._path):
            raise FileNotFoundError("No such file: config.ini")
            
        # 初始化配置文件对象
        self._config = configparser.ConfigParser()
        self._config.read(self._path, encoding='utf-8-sig')
        self._configRaw = configparser.RawConfigParser()
        self._configRaw.read(self._path, encoding='utf-8-sig')

    def get(self, section, name):
        return self._config.get(section, name)

    def getRaw(self, section, name):
        return self._configRaw.get(section, name)
    # 写回配置文件
    def setRaw(self, section, name, value):
        self._config.set(section, name, value)
        fp = open(self._path,'w')# config.write必须要一个fileobject对象
        self._config.write(fp)

global_config = Config()
```

##  配置文件的写法

#### 配置文件的语法要更自由

对于可实现同样功能的配置文件和Python源文件是有很大的不同的。 首先，配置文件的语法要更自由些，下面的赋值语句是等效的：

```python
prefix=/usr/local
prefix: /usr/local
```

#### 配置文件中的名字是不区分大小写的。例如：

```python
>>> cfg.get('installation','PREFIX')
'/usr/local'
>>> cfg.get('installation','prefix')
'/usr/local'
>>>
```

#### 在解析值的时候，`getboolean()` 方法查找任何可行的值。例如下面都是等价的：

```python
log_errors = true
log_errors = TRUE
log_errors = Yes
log_errors = 1
```

#### 它并不是从上而下的顺序执行。 文件是安装一个整体被读取的。

或许配置文件和Python代码最大的不同在于，它并不是从上而下的顺序执行。 文件是安装一个整体被读取的。如果碰到了变量替换，它实际上已经被替换完成了。 例如，在下面这个配置中，`prefix` 变量在使用它的变量之前或之后定义都是可以的：

```
[installation]
library=%(prefix)s/lib
include=%(prefix)s/include
bin=%(prefix)s/bin
prefix=/usr/local
```

#### 它能一次读取多个配置文件然后合并成一个配置。 

`ConfigParser` 有个容易被忽视的特性是它能一次读取多个配置文件然后合并成一个配置。 例如，假设一个用户像下面这样构造了他们的配置文件：

```python
; ~/.config.ini
[installation]
prefix=/Users/beazley/test

[debug]
log_errors=False
```

读取这个文件，它就能跟之前的配置合并起来。如：

```python
>>> # Previously read configuration
>>> cfg.get('installation', 'prefix')
'/usr/local'

>>> # Merge in user-specific configuration
>>> import os
>>> cfg.read(os.path.expanduser('~/.config.ini'))
['/Users/beazley/.config.ini']

>>> cfg.get('installation', 'prefix')
'/Users/beazley/test'
>>> cfg.get('installation', 'library')
'/Users/beazley/test/lib'
>>> cfg.getboolean('debug', 'log_errors')
False
>>>
```

#### 变量的改写采取的是后发制人策略，以最后一个为准。

仔细观察下 `prefix` 变量是怎样覆盖其他相关变量的，比如 `library` 的设定值。 产生这种结果的原因是变量的改写采取的是后发制人策略，以最后一个为准。 你可以像下面这样做试验：

```
>>> cfg.get('installation','library')
'/Users/beazley/test/lib'
>>> cfg.set('installation','prefix','/tmp/dir')
>>> cfg.get('installation','library')
'/tmp/dir/lib'
>>>
```

最后还有很重要一点要注意的是Python并不能支持.ini文件在其他程序（比如windows应用程序）中的所有特性。 确保你已经参阅了configparser文档中的语法详情以及支持特性。



## 配置文件写回到文件

```python
def setRaw(self, section, name, value):
    self._config.set(section, name, value)
    fp = open(self._path,'w')# config.write必须要一个fileobject对象
    self._config.write(fp)



from configparser import ConfigParser
cfg = ConfigParser()
cfg.read('config.ini')
#['config.ini']
cfg.sections()
#['url', 'package']
cfg.set('server','port','9000') # config中没有这个sections的时候，这样会报错
"""Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/configparser.py", line 1201, in set
    super().set(section, option, value)
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/configparser.py", line 902, in set
    raise NoSectionError(section) from None
configparser.NoSectionError: No section: 'server'
"""
cfg.set('url','port','9000')
cfg.write(sys.stdout)  # 写到sys默认输出
"""[url]
pymysql = https://pypi.org/project/pymysql/#files
port = 9000
[package]
pg = [django,scrapy]
"""
cfg.set('url','port','8000')
cfg.write(sys.stdout) # 输出到控制台

"""
[url]
pymysql = https://pypi.org/project/pymysql/#files
port = 8000
[package]
pg = [django,scrapy]
"""

```

