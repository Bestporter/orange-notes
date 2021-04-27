# form组件

```python
"""
注册功能：
	获取用户名和密码，利用form表单提交数据，在后端判断用户名和密码是否符合一定的条件：
		用户名不能包含xxx
		密码不能少于三位
如符合条件需要你将提示信息展示到前端页面
"""

class MyForm(forms.Form):
    username = forms.CharField(min_length=3,max_length=5)
    password = forms.CharField(min_length=3,max_length=5)
    email = forms.EmailField()
```

forms中的方法

```python
form_obj.cleaned_data # 查看所有校验通过的数据
form_obj.errors # 查看不通过的数据及原因
form
```

Forms 中的钩子函数（HOOK）

```python
"""
在特定的节点自动触发完成响应操作

钩子函数在forms组件中就类似于第二道关卡，能够让我们自定义校验规则

在forms组件中有两类钩子
	1.局部钩子
		拿到的数据得返回回去
	2.全局钩子

"""
```

源码：切入点是is_valid()函数

```python
def is_valid(self):
  	"""Return True if the form has no errors, 
  	or False otherwise."""
    return self.is_bound and not self.errors
	# 返回True 需要self.is_bound True  and self.errors False

# 只要传了值就是True  
self.is_bound = data is not None or files is not None
# 默认为none
self._errors = None  # Stores the errors after clean() has been called.


@property
def errors(self):
        """Return an ErrorDict for the data provided for the form."""
    if self._errors is None:
      # 肯定会进来这里，因为初始化self._errors赋值了None
      	self.full_clean()
    return self._errors
  
# forms组件 精髓
def full_clean(self):
    """
        Clean all of self.data and populate self._errors and self.cleaned_data.
    """
    self._errors = ErrorDict()# 就是一个字典，后缀Dict一般都继承字典
    
    if not self.is_bound:  # Stop further processing.
      # 当数据的时候肯定不会进
      return
    
    self.cleaned_data = {}
    # If the form is permitted to be empty, and none of the form data has
    # changed from the initial data, short circuit any validation.
    # 不会执行
    if self.empty_permitted and not self.has_changed():
      return
		
    # 整个forms的功能所在
    self._clean_fields()  # 校验字段 + 局部钩子
    self._clean_form() # 全局钩子
    self._post_clean() # 内部的钩子
    
    
def _clean_fields(self):
  	# 循环获取字段名和字段对象
    for name, field in self.fields.items(): 
        # value_from_datadict() gets the data from the data dictionaries.
        # Each widget type knows how to retrieve its own data, because some
        # widgets split data over several HTML fields.
        # 校验字段
        if field.disabled:
            value = self.get_initial_for_field(field, name)
        else:          	
          	# 获取字段对应的数据
            value = field.widget.value_from_datadict(self.data, self.files, self.add_prefix(name))# 类似于反射
        try:
          	
            if isinstance(field, FileField):
                initial = self.get_initial_for_field(field, name)
                value = field.clean(value, initial)
            else:
                value = field.clean(value)
            self.cleaned_data[name] = value # 将合法字段添加进去
            # 反射获取局部钩子函数
            if hasattr(self, 'clean_%s' % name):						
              	# 调用局部钩子函数，需要返回值
                # 出错可以抛出异常，因为这里有异常捕获，也可以直接用add_error添加错误信息。
                value = getattr(self, 'clean_%s' % name)() 
                # 这里说明了局部钩子函数需要返回修改了的值
                self.cleaned_data[name] = value
        except ValidationError as e:
            self.add_error(name, e) # 添加提示信息
            

def _clean_form(self):
    try:
        cleaned_data = self.clean()  # 由此，全局钩子需要返回值
    except ValidationError as e:
        self.add_error(None, e)
    else:
        if cleaned_data is not None:
            self.cleaned_data = cleaned_data
```

