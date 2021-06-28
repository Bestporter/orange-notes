通配符：
* 匹配任意个字符
? 匹配单个任意字符

# 全匹配
[root@localhost ~]# mv test/* dev
# 后缀匹配
[root@localhost ~]# mv dev/*.txt test/
# 前缀匹配
[root@localhost ~]# mv jie* dev
# 包含匹配
[root@localhost ~]# mv *ea* dev

# 将匹配如 a.txt 不会匹配 ab.txt 。? 不是可选，而是表示任意单个字符
[root@localhost ~]# mv test/a?.txt dev

[root@localhost ~]# mv *a??.txt dev

# 可以指定带通配符的多个源
[root@localhost ~]# mv dev/* test/* ./
