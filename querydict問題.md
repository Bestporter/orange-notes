```python
odep_invs=DepartmentInventory.objects.filter(department=department,)
for item in odep_invs:
  item['updateTime'] = item['updateTime'].strftime("%Y-%m-%d  %H:%M:%S")
  """
  报错DepartmentInventory   不能subscriptable
  因为filter返回的是DepartmentInventory对象列表
  """
  #添加values
 odep_invs=DepartmentInventory.objects.filter(department=department,).values(*field)
# 取得的是querydict对象
```

