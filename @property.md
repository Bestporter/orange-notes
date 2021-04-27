### @property



property属性的定义和调用要注意一下几点：

1. 定义时，在实例方法的基础上添加 @property 装饰器；并且**仅有一个self参数**

2. 调用时，无需括号

#### 经典类

```python
# ############### 定义 ###############
class Goods:
    @property
    def price(self):
        return "laowang"


# ############### 调用 ###############
obj = Goods()
result = obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
print(result)
```

#### 新式类

```python
#coding=utf-8
# ############### 定义 ###############
class Goods:
    """python3中默认继承object类
        以python2、3执行此程序的结果不同，因为只有在python3中才有@xxx.setter  @xxx.deleter
    """

    @property
    def price(self):
        print('@property')

    @price.setter
    def price(self, value):
        print('@price.setter')

    @price.deleter
    def price(self):
        print('@price.deleter')


# ############### 调用 ###############
obj = Goods()
obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
obj.price = 123  # 自动执行 @price.setter 修饰的 price 方法，并将  123 赋值给方法的参数
del obj.price  # 自动执行 @price.deleter 修饰的 price 方法

```

注意：

- **经典类中的属性只有一种访问方式，其对应被 @property 修饰的方法**
- **新式类中的属性有三种访问方式，并分别对应了三个被 @property、@方法名.setter、@方法名.deleter 修饰的方法**

由于新式类中具有三种访问方式，我们可以根据它们几个属性的访问特点，分别**将三个方法定义为对同一个属性：获取、修改、删除**

```python
class Goods(object):
    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    @property
    def price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price
		# 修改
    @price.setter
    def price(self, value):
        self.original_price = value
		# 删除
    @price.deleter
    def price(self):
        print('del')
        del self.original_price


obj = Goods()
print(obj.price)  # 获取商品价格
```

**当使用类属性的方式创建property属性时，经典类和新式类无区别**

```python
class Goods(object):
    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    def get_price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    def set_price(self, value):
        self.original_price = value

    def del_price(self):
        del self.original_price

    PRICE = property(get_price, set_price, del_price, '价格属性描述...')


obj = Goods()
obj.PRICE  # 获取商品价格
```

综上所述:

- 定义property属性共有两种方式，分别是【装饰器】和【类属性】，而【装饰器】方式针对经典类和新式类又有所不同。
- 通过使用property属性，能够简化调用者在获取数据的流程