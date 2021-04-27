

#### 本地yum配置

1）`mount /dev/cdrom /mnt/rep` 首先将本地磁盘镜像挂载到/mnt 目录下（临时挂载）

2）`touch /etc/yum.repos.d/centos.repo` 创建yum源的配置文件

3）`vim /etc/yum.repos.d/centos.repo` 配置yum源

```shell
name=centos #使用户更容易的读懂文件。

baseurl=file:///mnt/rep/ #我们指定的查找依赖关系软件的路径

enabled=1 #0表示baseurl定义的路径是不可用的，1表示定义的路径是可用的。

gpgcheck=0 #进行gpg检测，0表示不进行，1表示进行
```

服务器网址 `10.134.82.189`

#### 常用的命令

```
yum repolist 查看软件包的数量

yum clean all 清楚缓存

yum provides */命令 根据命令查找软件包

yum -y insatll 软件包 软件包安装
```

