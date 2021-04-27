### getattr用法

```python
def process_response(self, request, response):
    # Don't set it if it's already in the response
    if response.get('X-Frame-Options') is not None:
        return response

    # getattr
    if getattr(response, 'xframe_options_exempt', False):
        return response

    response['X-Frame-Options'] = self.get_xframe_options_value(request,                                                             response)
    return response
```

