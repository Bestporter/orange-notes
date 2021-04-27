## python中的反射

###### 1.什么是反射：

```
通过字符串映射object对象的方法或者属性
```

###### 2.反射的方法

```
hasattr(obj,name_str): 判断objec是否有name_str这个方法或者属性
getattr(obj,name_str): 获取object对象中与name_str同名的方法或者函数
setattr(obj,name_str,value): 为object对象设置一个以name_str为名的value方法或者属性
delattr(obj,name_str): 删除object对象中的name_str方法或者属性
```

```
#  场景: 当用户在操作时,面对用户不同的操作就需要调用不同的函数
#  如果用if,elif语句的话,会存在一个问题,当用户有500个不同的操作,
#  就需要写500次if,elif。这样就会出现代码重复率高,而且还不易维护
#  这时候反射就出现了,通过字符串映射对象或方法

# 实例化一个对象: 胖毛
c = User('胖毛')

# 用户输入指令
choose = input('>>>:')

# 通过hasattr判断属性/方法是否存在
if hasattr(c,choose):
    # 存在,用一个func接收
    func = getattr(c,choose)
    # 通过类型判断是属性还是方法
    if type(func) == str:
        print(func)
    else:        
    		func()
else:
    print('操作有误,请重新输入')
```

