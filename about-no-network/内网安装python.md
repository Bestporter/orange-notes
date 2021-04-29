内网安装python

[参考网址](https://blog.csdn.net/weixin_42414714/article/details/114193075)

1.   切换到项目的本地虚拟环境，导出模块清单：

```python
pip3 freeze > requirements.txt
```

2.  下载requirements.txt中的安装包

```python
# YOUR_DIR为你要存放安装包的文件夹路径
pip3 --timeout=600 wheel -i https://pypi.tuna.tsinghua.edu.cn/simple -w YOUR_DIR -r requirements.txt
pip3 --timeout=600 download -i https://pypi.tuna.tsinghua.edu.cn/simple -d YOUR_DIR -r requirements.txt
```

注意：

（1）两个命令都要执行，目的是为了防止遗漏；

（2）生产环境一般为Linux，因此也要在linux上进行上述操作，如果是Mac或者Windows，下载的安装包可能是不能用的；

（3）有一个第三模块virtualenv是必须的，如果清单中没有，需要手动下载。

```console
#### 解压
$ tar xvJf Python-3.6.5.tgz

#### 进入目录
cd Python-3.6.5

#### 新建文件夹
$ mkdir bld
$ cd bld/

#### 生成Makefile（--prefix参数指定程序安装目录）
$ ../configure --prefix=/usr/local/python3
#### 注：指定安装目录的目的是为了方便以的维护、卸载或移植。当不再需要时，删除该安装目录，就可以把软件卸载干净

#### 编译
$ make all

#### 安装
make install

#### 查看安装后的位置
$ whereis python3
```

