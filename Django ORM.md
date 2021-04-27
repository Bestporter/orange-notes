annotate()方法，分组（group by）

```python
# 按学生分组，统计每个学生的爱好数量
Student.objects.annotate(Count('hobbies'))

# 按学生分组，统计每个学生爱好数量，并自定义字段名
Student.objects.annotate(hobby_count_by_student=Count('hobbies'))
```

```python
# 先按爱好分组，再统计每组学生数量, 然后筛选出学生数量大于1的爱好。
Hobby.objects.annotate(student_num=Count('student')).filter(student_num__gt=1)
```

```python
# 先按爱好分组，筛选出以'd'开头的爱好，再统计每组学生数量。
Hobby.objects.filter(name__startswith="d").annotate(student_num=Count('student‘))

# 先按爱好分组，再统计每组学生数量, 然后按每组学生数量大小对爱好排序。
Hobby.objects.annotate(student_num=Count('student')).order_by('student_num')



# 统计最受学生欢迎的5个爱好。
Hobby.objects.annotate(student_num=Count('student')).order_by('-student_num')[:5]

# 按学生名字分组，统计每个学生的爱好数量。
Student.objects.values('name').annotate(Count('hobbies'))

# 按学生名字分组，统计每个学生的爱好数量。
Student.objects.annotate(hobby_count=Count('hobbies')).values('name', 'hobby_count')
```

```mysql
ret = Publish.objects.annotate(a=Avg('book__price'))
#对应的SQL语句
SELECT
	`app01_publish`.`id`,
	`app01_publish`.`name`,
	`app01_publish`.`city`,
	`app01_publish`.`email`,
	AVG( `app01_book`.`price` ) AS `a` 
FROM
	`app01_publish`
	LEFT OUTER JOIN `app01_book` ON ( `app01_publish`.`id` = `app01_book`.`publish_id` ) 
GROUP BY
	`app01_publish`.`id` 
ORDER BY
NULL

```

# 获取ORM查询语句对应的SQL

通过queryset.query获取对应的SQL

```python
>>> queryset = Event.objects.all()
>>> str(queryset.query)
SELECT "events_event"."id", "events_event"."epic_id",
    "events_event"."details", "events_event"."years_ago"
    FROM "events_event"
```

# OR查询

常见的 `OR` filtering with two ore more conditions. 例如查找users with firstname starting with ‘R’ and last_name starting with ‘D’.

**Django provides two options.**

- `queryset_1 | queryset_2`
- `filter(Q(<condition_1>)|Q(<condition_2>)`

```python
queryset = User.objects.filter(
        first_name__startswith='R'
    ) | User.objects.filter(
    last_name__startswith='D'
)
queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
 # 或者使用Q objects
from django.db.models import Q
qs = User.objects.filter(Q(first_name__startswith='R')|Q(last_name__startswith='D'))
```

```python
ret = Publish.objects.filter(id=1) | ublish.objects.filter(id=2)
```

```mysql
#对应的SQL是
SELECT
	`app01_publish`.`id`,
	`app01_publish`.`name`,
	`app01_publish`.`city`,
	`app01_publish`.`email` 
FROM
	`app01_publish` 
WHERE
	(
		`app01_publish`.`id` = 1 
	OR `app01_publish`.`id` = 2 
	)
```

# AND查询

Say you want to find users with `firstname` starting with ‘R’ AND `last_name` starting with ‘D’.

**Django provides three options.**

- `filter(<condition_1>, <condition_2>)`
- `queryset_1 & queryset_2`
- `filter(Q(<condition_1>) & Q(<condition_2>))`

```python
# 以下个表达式生成的SQL是完全一样的
queryset_1 = User.objects.filter(first_name__startswith='R',
    								last_name__startswith='D')
queryset_2 = User.objects.filter(first_name__startswith='R') & User.objects.filter(last_name__startswith='D')
queryset_3 = User.objects.filter(Q(first_name__startswith='R') &
    Q(last_name__startswith='D'))
```

# NOT查询

Say you want to fetch all users with id NOT < 5. You need a NOT operation.

**Django provides two options.**

- `exclude(<condition>)`
- `filter(~Q(<condition>))`

```mysql
SELECT id, username, first_name, last_name, email FROM auth_user WHERE NOT id < 5;
```

```python
# 查询id大于等于5的所有user
>>> from django.db.models import Q
>>> queryset = User.objects.filter(~Q(id__lt=5))
```

# UNION查询

The union operation can be performed **only with the querysets having same fields and the datatypes.** Hence our last union operation encountered error. You can do a union on two models as long as they have same fields or same subset of fields.

```python
>>> q3 = EventVillain.objects.all()
>>> q3
<QuerySet [<EventVillain: EventVillain object (1)>]>
>>> q1.union(q3)
django.db.utils.OperationalError: SELECTs to the left and right of UNION do not have the same number of result columns
```

在进行union查询之前，可以先对fields进行筛选然后再union

Since `Hero` and `Villain` both have the `name` and `gender`, we can use `values_list` to limit the selected fields then do a union.

```python
Hero.objects.all().values_list(
    "name", "gender"
).union(
Villain.objects.all().values_list(
    "name", "gender"
))
```

```mysql
(SELECT `app01_publish`.`name` FROM `app01_publish`) UNION (SELECT `app01_author`.`name` FROM `app01_author`)
```

```python
ret = Publish.objects.all().values('name')
rest = Author.objects.all().values('name')
s = ret.union(rest)
```

# 仅在queryset中选择某些字段

 But sometimes, you do not need to use all the fields. In such situations, we can query only desired fields.

**Django provides two ways to do this**

- values and values_list methods on queryset.
- only_method

Say, we want to get `first_name` and `last_name` of all the users whose name starts with **R**. You do not want the fetch the other fields to reduce the work the DB has to do.

#### 第一种方法：

```python
>>> queryset = User.objects.filter(
    first_name__startswith='R'
).values('first_name', 'last_name')
>>> queryset
<QuerySet [{'first_name': 'Ricky', 'last_name': 'Dayal'}, {'first_name': 'Ritesh', 'last_name': 'Deshmukh'}, {'first_name': 'Radha', 'last_name': 'George'}, {'first_name': 'Raghu', 'last_name': 'Khan'}, {'first_name': 'Rishabh', 'last_name': 'Deol'}]
```

#### 第二种

```python
>>> queryset = User.objects.filter(
    first_name__startswith='R'
).only("first_name", "last_name")
```



The only difference between `only` and `values` is `only` **also fetches the `id`.**

```python
ret = Publish.objects.filter(name__startswith='华').only('name')
print(ret.query)
ret = Publish.objects.filter(name__startswith='华').values('name')
print(ret.query)
    """
SELECT `app01_publish`.`id`, `app01_publish`.`name` FROM `app01_publish` WHERE `app01_publish`.`name` LIKE BINARY 华%

SELECT `app01_publish`.`name` FROM `app01_publish` WHERE `app01_publish`.`name` LIKE BINARY 华%
    """
```

# 子查询 subquery

```python
>>> from django.db.models import Subquery
>>> users = User.objects.all()
>>> UserParent.objects.filter(user_id__in=Subquery(users.values('id')))
<QuerySet [<UserParent: UserParent object (2)>, <UserParent: UserParent object (5)>, <UserParent: UserParent object (8)>]>

class Category(models.Model):
    name = models.CharField(max_length=100)


class Hero(models.Model):
    # ...
    name = models.CharField(max_length=100)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)

    benevolence_factor = models.PositiveSmallIntegerField(
        help_text="How benevolent this hero is?",
        default=50
    )
 
hero_qs = Hero.objects.filter(
    category=OuterRef("pk")
).order_by("-benevolence_factor")
Category.objects.all().annotate(
    most_benevolent_hero=Subquery(
        hero_qs.values('name')[:1]
    )
)
# 对应的Mysql语句
SELECT "entities_category"."id",
       "entities_category"."name",
  (SELECT U0."name"
   FROM "entities_hero" U0
   WHERE U0."category_id" = ("entities_category"."id")
   ORDER BY U0."benevolence_factor" DESC
   LIMIT 1) AS "most_benevolent_hero"
FROM "entities_category"

```

> We are ordering the `Hero` object by `benevolence_factor` in DESC order, and using `category=OuterRef("pk")` to declare that we will be using it in a subquery.
>
> Then we annotate with `most_benevolent_hero=Subquery(hero_qs.values('name')[:1])`, to get use the subquery with a `Category` queryset. The `hero_qs.values('name')[:1]` part picks up the first name from subquery.



### 如何根据比较查询集的字段值来筛选条件集

> What if you want to compare the first_name and last name? You can use the `F` object. Create some users first.

```python
In [27]: User.objects.create_user(email="shabda@example.com", username="shabda", first_name="Shabda", last_name="Raaj")
Out[27]: <User: shabda>

In [28]: User.objects.create_user(email="guido@example.com", username="Guido", first_name="Guido", last_name="Guido")
Out[28]: <User: Guido>
In [29]: User.objects.filter(last_name=F("first_name"))
Out[29]: <QuerySet [<User: Guido>]>
```

>  `F` also works with calculated field using annotate. What if we wanted users whose first and last names have same letter?

```
In [41]: User.objects.create_user(email="guido@example.com", username="Tim", first_name="Tim", last_name="Teters")
Out[41]: <User: Tim>
#...
In [46]: User.objects.annotate(first=Substr("first_name", 1, 1), last=Substr("last_name", 1, 1)).filter(first=F("last"))
Out[46]: <QuerySet [<User: Guido>, <User: Tim>]>
```

`F` can also be used with `__gt`, `__lt` and other expressions.

## 将多个查询结果转换为字典列表

data = entity.objects.all().values()

data_dict_list = list(data)

### 查詢有重复数据的记录

Say you want all users whose `first_name` matches another user.

You can find duplicate records using the technique below.

```python
>>> duplicates = User.objects.values(
 'first_name').annotate(name_count=Count('first_name')).filter(name_count__gt=1)
>>> duplicates
<QuerySet [{'first_name': 'John', 'name_count': 3}]>
```

If you need to fill all the records, you can do

```python
>>> records = User.objects.filter(first_name__in=[item['first_name'] for item in duplicates])
>>> print([item.id for item in records])
[2, 11, 13]
```

### 查询没有重复的数据的记录

```python
distinct = User.objects.values(
    'first_name'
).annotate(
    name_count=Count('first_name')
).filter(name_count=1)
records = User.objects.filter(first_name__in=[item['first_name'] for item in distinct])
```

This is different from `User.objects.distinct("first_name").all()`, which will pull up the first record when it encounters a distinct `first_name`.

### Q查询 or, and , not

If you want to `OR` your conditions.

```python
>>> from django.db.models import Q
>>> queryset = User.objects.filter(
    Q(first_name__startswith='R') | Q(last_name__startswith='D')
)
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
```

If you want to `AND` your conditions.

```python
>>> queryset = User.objects.filter(
    Q(first_name__startswith='R') & Q(last_name__startswith='D')
)
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: rishab>]>
```

If you want to find all users whose `first_name` starts with ‘R’, but not if the `last_name` has ‘Z’

```python
>>> queryset = User.objects.filter(
    Q(first_name__startswith='R') & ~Q(last_name__startswith='Z')
)
```

