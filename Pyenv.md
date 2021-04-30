Pyenv

[参考文章](https://einverne.github.io/post/2017/04/pyenv.html)

因为国内网络环境，如果在局域网内下载 pip 慢，可以尝试使用 aliyun 提供的镜像，创建 `vim ~/.pip/pip.conf` ，然后填入：

```
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

直接 clone pyenv 项目到 HOME 根目录。 建议路径为：$HOME/.pyenv

```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```