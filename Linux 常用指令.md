Linux 常用指令

## 查看shell

**查看系统支持的shell： `cat  /etc/shells`**

**查看现在使用的shell：** `echo $SHELL `或者 `env | grep SHELL` 或者`echo $0`

修改默认shell  `chsh -s /bin/bash`

## linux下查看软连接的信息

```python
$ readlink -f python
/usr/local/python3/bin/python3.6
```

## linux启动httpd

