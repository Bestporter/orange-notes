

​	下面使用这种方法可以少用一次else

```python
class MiddlewareMixin:
    def __init__(self, get_response=None):
        self.get_response = get_response
        super().__init__()

    def __call__(self, request):
        response = None
        if hasattr(self, 'process_request'):
            response = self.process_request(request)
        # 返回最后一个真值，赋值给response
        response = response or self.get_response(request)
        if hasattr(self, 'process_response'):
            response = self.process_response(request, response)
        return response
```



```python
>>> ‘a’ or ‘b’  
’a’  
>>> ” or ‘b’  
’b’  
>>> ” or [] or{}  
{}  
>>> ‘a’ and ‘b’  
’b’  
>>> ” and ‘b’  
”  
>>> ’a’ and ‘b’ and ‘c’  
’c’  
>>> a = “betabin”  
>>> b = ”python”  
>>> 1 and a or b  
’betabin’  
>>> 0 and a or b  
’python’  
```

