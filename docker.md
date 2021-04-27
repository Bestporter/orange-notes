# docker 

[参考](http://www.liuqingzheng.top/linux/Docker/3-Docker%E4%B9%8B%E9%95%9C%E5%83%8F/)

## run

[参考书籍](https://yeasy.gitbook.io/docker_practice/container/run)

**docker run**的**--rm**选项详解 在**Docker**容器退出时，默认容器内部的文件系统仍然被保留，以方便调试并保留用户数据。 注意，--**rm**选项也会清理容器的匿名data volumes。 所以，执行**docker run**命令带**--rm**命令选项，等价于在容器退出后，执行**docker rm** -v。

我们在Server机器上搭建私有仓库，一条命令即可，非常简单

```
$ docker run --name docker-registry -d -p 5000:5000 --restart=always registry:2
```

> --name 用来设置容器的名字
>
> -d 后台启动
>
> -p 5000:5000 指定宿主机5000端口映射容器5000端口，
>
> --restart=always Docker容器的重启策略

Docker容器的重启策略是面向生产环境的一个启动策略

>  no，默认策略，在容器退出时不重启容器
>
>   on-failure，在容器非正常退出时（退出状态非0），才会重启容器
>
>   on-failure:3，在容器非正常退出时重启容器，最多重启3次
>
>   always，在容器退出时总是重启容器
>
>   unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

```shell
$ docker run -t -i ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
```

`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。TTY是由虚拟控制台，串口以及伪终端设备组成的终端设备。

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载

- 利用镜像创建并启动一个容器

- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层

- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去

- 从地址池配置一个 ip 地址给容器

- 执行用户指定的应用程序

- 执行完毕后容器被终止

- 可以使用 `docker container stop` 来终止一个运行中的容器。

- `docker container stop(restart/start) 07cedcb90070`

- 可以利用 `docker container start` 命令，直接将一个已经终止的容器启动运行。

- 终止状态的容器可以用 `docker container ls -a` 命令看到。

- 此外，`docker container restart` 命令会将一个运行态的容器终止，然后再重新启动它。

  容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 `ps` 或 `top` 来查看进程信息。

### 守护态运行 -d 



更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

```python
# 使用-d
f7691915$ docker run -d  ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
f98f51066637a1d0ea1be35db68709a2dbabe30e12e0021416c3875f807f167b
# 不使用-d
f7691915$ docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
```

使用`-d`参数，此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面(输出结果可以用 `docker logs f98f51066637a1d0ea1be35db68709a2dbabe30e12e0021416c3875f807f167b ([container])` 查看)。

**注：** 容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker container ls` 命令来查看容器信息。

## 进入容器（-d容器进入后台的时候，如何进入容器操作）

#### docker attach

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

```shell
# docker container ls查出 container_id 
f7691915$ docker attach 3eb3
hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world
# 注意： 如果从这个 stdin 中 exit，会导致容器的停止。
```

## exec 命令

### -i -t 参数

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。



```shell
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

$ docker exec -i 69d1 bash   # 只用-i看不到终端
ls
bin
boot
dev
...
$ docker exec -it 69d1 bash  # 用-it 能看到终端
root@69d137adef7a:/#
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

## 导入和导出

### 导出容器

```shell
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:18.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar  # 导出到当前目录下的
# 这样将导出容器快照到本地文件。
```

### 导入容器快照

`docker import `

直接使用`docker import xxx.tar`  REPOSITORY和tag将会是none 

所以要通过stdin导入

```shell
$ cat exampleimage.tgz | docker import - exampleimagelocal:new
$ cat exampleimage.tgz | docker import --message "New image imported from tarball" - exampleimagelocal:new


$ cat ubuntu.tar | docker import test/ubuntu:v1.0
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
# 也可以通过指定 URL 或者某个目录来导入
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

*注：用户既可以使用* *`docker load`* *来导入镜像存储文件到本地镜像库，也可以使用* *`docker import`* *来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*

## 删除

### 删除容器

可以使用 `docker container rm` 来删除一个处于终止状态的容器。例如

```shell
$ docker container rm  trusting_newton
trusting_newton
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。

### 清理所有处于终止状态的容器

用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```
$ docker container prune
```

## docker image

查`docker image ls`

删除`docker image rm [repository]:[tag]`或者`docker rmi [options] image [image……]. 

(option) -f --force 强制删除

## 镜像备份与导入

将本地的一个或者多个镜像打包保存成本地tar文件，docker save， -o指定写入的文件名和路径

```shell
# 命令演示：
docker save -o linux_images.tar centos ubuntu
# 导入 docker load 
# 作用：
	将save命令打包的镜像导入本地镜像库中
# 命令格式：
	docker load [OPTIONS]
# 命令参数(OPTIONS)：	
	-i,  --input string   	指定要打入的文件，如没有指定，默认是STDIN
	-q, --quiet          		不打印导入过程信息
# 命令演示
docker load -i linux_images.tar
docker load -i linux_images.tar -q

```

### 镜像重命名 - docker tag

```shell
# 作用：
	对本地镜像的NAME、TAG进行重命名，并新产生一个命名后镜像,注意。原来的名字的镜像依然存在
# 命令格式：
	docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
# 命令参数(OPTIONS)：	
	无
# 命令演示
docker tag e934 centos-newname:newtag
```

###  镜像详细信息 – docker image inspect/docker inspect

```
# 作用：
	查看本地一个或多个镜像的详细信息
# 命令格式：
	docker image inspect [OPTIONS] IMAGE [IMAGE...]
      或者 docker inspect [OPTIONS] IMAGE [IMAGE...]
# 命令参数(OPTIONS)：	
	-f, --format string          利用特定Go语言的format格式输出结果
# 命令演示：
docker image inspect -f "{{json .id}}" centos
docker image inspect -f "{{json .Created}}" centos
docker image inspect
```

### 镜像历史信息 – docker history

```
# 作用：
	查看本地一个镜像的历史(历史分层)信息
# 命令格式：
	docker history [OPTIONS] IMAGE
# 命令参数(OPTIONS)：
	-H, --human		将创建时间、大小进行优化打印(默认为true)
	-q, --quiet           	只显示镜像ID
	     --no-trunc        	不缩略显示
# 命令演示
docker history ubuntu
docker history ubuntu -H=false
```

## 容器提交 docker commit

```shell
# 作用：
	根据容器生成一个新的镜像
# 命令格式：
	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
# 命令参数(OPTIONS)：
	-a, --author string    	作者
	-c, --change list      	为创建的镜像加入Dockerfile命令
	-m, --message string   	提交信息，类似git commit -m
	-p, --pause            	提交时暂停容器 (default true)
# 命令演示
docker run --rm -dti centos bash
docker exec -d 容器id号 yum -y install net-tools
docker commit -m 'install net-tools' 容器id号 centos-net-tools:lastest
docker images

docker history centos-net-tools
```

## 容器导出 docker export

```python
# 作用：
	将容器当前的文件系统导出成一个tar文件
# 命令格式：
	docker export [OPTIONS] CONTAINER
# 命令参数(OPTIONS)：
	-o, --output string   		指定写入的文件，默认是STDOUT

```

## 仓库（Repository）

另外，在查找的时候通过 `--filter=stars=N` 参数可以指定仅显示收藏数量为 `N` 以上的镜像。

```shell
$ docker search centos   # DESCRIPTION 中说明是The official build of CentOS. 官方镜像
# 有用户名前缀的是用户创造的
NAME                                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                                          The official build of CentOS.                   465       [OK]
tianon/centos                                   CentOS 5 and 6, created using rinse instea...   28
blalor/centos                                   Bare-bones base CentOS 6.5 image                6                    [OK]
saltstack/centos-6-minimal                                                                      6                    [OK]
tutum/centos-6.4                                DEPRECATED. Use tutum/centos:6.4 instead. ...   5                    [OK]
```

### 推送镜像

```python
$ docker tag ubuntu:18.04 username/ubuntu:18.04

$ docker image ls

REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
ubuntu                                                   18.04                  275d79972a86        6 days ago          94.6MB
username/ubuntu                                          18.04                  275d79972a86        6 days ago          94.6MB

$ docker push username/ubuntu:18.04

$ docker search username

NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
username/ubuntu
```

### 自动构建

自动构建（Automated Builds）功能对于需要经常升级镜像内程序来说，十分方便。

有时候，用户构建了镜像，安装了某个软件，当软件发布新版本则需要手动更新镜像。

而自动构建允许用户通过 Docker Hub 指定跟踪一个目标网站（支持 [GitHub](https://github.com/) 或 [BitBucket](https://bitbucket.org/)）上的项目，一旦项目发生新的提交 （commit）或者创建了新的标签（tag），Docker Hub 会自动构建镜像并推送到 Docker Hub 中。

要配置自动构建，包括如下的步骤：

- 登录 Docker Hub；
- 在 Docker Hub 点击右上角头像，在账号设置（Account Settings）中关联（Linked Accounts）目标网站；
- 在 Docker Hub 中新建或选择已有的仓库，在 `Builds` 选项卡中选择 `Configure Automated Builds`；
- 选取一个目标网站中的项目（需要含 `Dockerfile`）和分支；
- 指定 `Dockerfile` 的位置，并保存。

之后，可以在 Docker Hub 的仓库页面的 `Timeline` 选项卡中查看每次构建的状态。



## 私有仓库

#### 容器运行 docker-registry

获取官方registry



```shell
$ docker run -d -p 5000:5000 --restart=always --name registry registry

$ docker run -d \
    -p 5000:5000 \
    -v /tmp/data/registry:/var/lib/registry \
    registry
```

创建好私有仓库之后，就可以使用 `docker tag` 来标记一个镜像，然后推送它到仓库。例如私有仓库地址为 `127.0.0.1:5000`。

先在本机查看已有的镜像。`docker image ls`

使用 `docker tag` 将 `ubuntu:latest` 这个镜像标记为 `127.0.0.1:5000/ubuntu:latest`。

格式为 `docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]`。

然后用`docker push `上传标记的镜像。

用`curl`查看仓库中的镜像。

```shelll
$ curl 127.0.0.1:5000/v2/_catalog
{"repositories":["ubuntu"]}
```

# 注意事项

如果你不想使用 `127.0.0.1:5000` 作为仓库地址，比如想让本网段的其他主机也能把镜像推送到私有仓库。你就得把例如 `192.168.199.100:5000` 这样的内网地址作为私有仓库地址，这时你会发现无法成功推送镜像。

这是因为 Docker 默认不允许非 `HTTPS` 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制，或者查看下一节配置能够通过 `HTTPS` 访问的私有仓库。

## Ubuntu 16.04+, Debian 8+, centos 7

对于使用 `systemd` 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

注意：该文件必须符合 `json` 规范，否则 Docker 将不能启动。



## 私有仓库高级配置 （权限认证、安全传输层协议（TLS））

#### 第一步创建 `CA` 私钥。



```
$ openssl genrsa -out "root-ca.key" 4096
```

#### 第二步利用私钥创建 `CA` 根证书请求文件。



```
$ openssl req \
          -new -key "root-ca.key" \
          -out "root-ca.csr" -sha256 \
          -subj '/C=CN/ST=Guangdong/L=Shenzhen/O=Foxconn/CN=Foxconn Docker Registry CA'
```

> 以上命令中 `-subj` 参数里的 `/C` 表示国家，如 `CN`；`/ST` 表示省；`/L` 表示城市或者地区；`/O` 表示组织名；`/CN` 通用名称。

#### 第三步配置 `CA` 根证书，新建 `root-ca.cnf`

```
[root_ca]
basicConstraints = critical,CA:TRUE,pathlen:1
keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
subjectKeyIdentifier=hash
```

#### 第四步签发根证书

```
$ openssl x509 -req  -days 3650  -in "root-ca.csr" \
               -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
               -extfile "root-ca.cnf" -extensions \
               root_ca
```

#### 第五步生成站点 `SSL` 私钥

```
$ openssl genrsa -out "docker.domain.com.key" 4096
```

#### 第六步使用私钥生成证书请求文件

```
$ openssl req -new -key "docker.domain.com.key" -out "site.csr" -sha256 \
          -subj '/C=CN/ST=Guangdong/L=Shenzhen/O=Foxconn/CN=docker.domain.com'
```

#### 第七步配置证书，新建 `site.cnf` 文件

```
[server]
authorityKeyIdentifier=keyid,issuer
basicConstraints = critical,CA:FALSE
extendedKeyUsage=serverAuth
keyUsage = critical, digitalSignature, keyEncipherment
subjectAltName = DNS:docker.domain.com, IP:127.0.0.1
subjectKeyIdentifier=hash
```

#### 第八步签署站点 `SSL` 证书

```
$ openssl x509 -req -days 750 -in "site.csr" -sha256 \
    -CA "root-ca.crt" -CAkey "root-ca.key"  -CAcreateserial \
    -out "docker.domain.com.crt" -extfile "site.cnf" -extensions server
```

**这样已经拥有了 `docker.domain.com` 的网站 SSL 私钥 `docker.domain.com.key` 和 SSL 证书 `docker.domain.com.crt` 及 CA 根证书 `root-ca.crt`。**

新建 `ssl` 文件夹并将 `docker.domain.com.key` `docker.domain.com.crt` `root-ca.crt` 这三个文件移入，删除其他文件。





## 数据卷

### 创建一个数据卷

```shell
docker volume create my-vol
# 查看所有数据卷
docker volume ls
# 查看指定数据卷的信息
docker volume inspect my-vol
"""
[
    {
        "CreatedAt": "2021-01-27T11:13:53Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
"""
```

### 启动一个挂载数据卷的容器

在用 `docker run` 命令的时候，使用 `--mount` 标记来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。

下面创建一个名为 `web` 的容器，并加载一个 `数据卷` 到容器的 `/usr/share/nginx/html` 目录。

```shell
$ docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```

### 删除数据卷

```python
docker volume rm my-vol# 删除不掉的时候先删除container
docker container rm -v [id]
docker volume prune # 删除无主的数据卷

```

## 挂载主机目录

### 挂载一个主机目录作为数据卷

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

```python
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

上面的命令加载主机的 `/src/webapp` 目录到容器的 `/usr/share/nginx/html`目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 `--mount` 参数时如果本地目录不存在，Docker 会报错。

Docker 挂载主机目录的默认权限是 `读写`，用户也可以通过增加 `readonly` 指定为 `只读`

```python
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html:ro \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine
```

### 查看数据卷挂载主机目录的配置信息



```python 
docker inspect web

```

Mounts 中source中

---

### 挂载一个本地主机文件作为数据卷

`--mount` 标记也可以从主机挂载单个文件到容器中

```shell
$ docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
   bash

root@2affd44b4667:/# history
1  ls
2  diskutil list
```

这样就可以记录在容器输入过的命令了。

---

## 外部访问容器

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。

当使用 `-P` 标记时，Docker 会随机映射一个端口到内部容器开放的网络端口。

使用 `docker container ls` 可以看到，本地主机的 32768 被映射到了容器的 80 端口。此时访问本机的 32768 端口即可访问容器内 NGINX 默认页面。

```shell
$ docker run -d -P nginx:alpine

$ docker container ls -l
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
fae320d08268        nginx:alpine        "/docker-entrypoint.…"   24 seconds ago      Up 20 seconds       0.0.0.0:32768->80/tcp   bold_mcnulty
```

查看访问记录

```python
$ docker logs fae320
172.17.0.1 - - [25/Aug/2020:08:34:04 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:80.0) Gecko/20100101 Firefox/80.0" "-"
```

```
-p` 则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort
```

查看端口配置

```shell
$ docker port fa 80
0.0.0.0:32768
```

注意：

- 容器有自己的内部网络和 ip 地址（使用 `docker inspect` 查看，Docker 还可以有一个可变的网络配置。）
- `-p` 标记可以多次使用来绑定多个端口

例如

```shell
$ docker run -d \
    -p 80:80 \
    -p 443:443 \
    nginx:alpine
```

---

## 容器互联

如果你之前有 `Docker` 使用经验，你可能已经习惯了使用 `--link` 参数来使容器互联。

随着 Docker 网络的完善，强烈建议大家**将容器加入自定义的 Docker 网络来连接多个容器**，而不是使用 `--link` 参数。

---

### 新建网络

```shell
docker network create -d bridge my-net
```

`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay`。其中 `overlay` 网络类型用于Swarm mode，在本小节中你可以忽略它。

---

### 连接容器

运行一个容器，并联入my-net网络

```python
$ docker run -it --rm --name busybox1 --network my-net busybox sh
```

另开终端，再运行一个容器

```shell
$ docker run -it --rm --name busybox2 --network my-net busybox sh
```

然后互ping一下

---

### Docker compose

如果你有多个容器之间需要互相连接，推荐使用 [Docker Compose](https://yeasy.gitbook.io/docker_practice/compose)。

---

## 配置DNS

如何自定义配置容器的主机名和 DNS 呢？秘诀就是 Docker 利用虚拟文件来挂载容器的 3 个相关配置文件。

在容器中使用 `mount` 命令可以看到挂载信息：

```shell
$ mount
```

这种机制可以让宿主主机 DNS 信息发生更新后，所有 Docker 容器的 DNS 配置通过 `/etc/resolv.conf` 文件立刻得到更新。

配置全部容器的 DNS ，也可以在 `/etc/docker/daemon.json` 文件中增加以下内容来设置。

```shell
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```

这样每次启动的容器 DNS 自动配置为 `114.114.114.114` 和 `8.8.8.8`。使用以下命令来证明其已经生效。

```shell
$ docker run -it --rm ubuntu:18.04  cat etc/resolv.conf

nameserver 114.114.114.114
nameserver 8.8.8.8
```

如果用户想要手动指定容器的配置，可以在使用 `docker run` 命令启动容器时加入如下参数：

`-h HOSTNAME` 或者 `--hostname=HOSTNAME` 设定容器的主机名，它会被写到容器内的 `/etc/hostname` 和 `/etc/hosts`。但它在容器外部看不到，既不会在 `docker container ls` 中显示，也不会在其他的容器的 `/etc/hosts` 看到。

`--dns=IP_ADDRESS` 添加 DNS 服务器到容器的 `/etc/resolv.conf` 中，让容器用这个服务器来解析所有不在 `/etc/hosts` 中的主机名。

`--dns-search=DOMAIN` 设定容器的搜索域，当设定搜索域为 `.example.com` 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 `host.example.com`。

> 注意：如果在容器启动时没有指定最后两个参数，Docker 会默认用主机上的 `/etc/resolv.conf` 来配置容器。

---

## docker compose

`Compose` 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

`Compose` 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 `Compose` 来进行编排管理。

