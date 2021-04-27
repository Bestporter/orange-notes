# dis库

[官方文档](https://docs.python.org/zh-cn/3/library/dis.html)

```python
import dis

def main():
    from collections import defaultdict
    x = defaultdict(int)
    print(x[1])
dis.dis(main)
```

