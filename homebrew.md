```bash



# 中科大
HOMEBREW_CORE_GIT_REMOTE=https://mirrors.ustc.edu.cn/homebrew-core.git
/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"
/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"


```

https://zhuanlan.zhihu.com/p/90508170。  参考网址



提前设置`homebrew-core`镜像源并执行：

```bash
# 中科大
HOMEBREW_CORE_GIT_REMOTE=https://mirrors.ustc.edu.cn/homebrew-core.git

/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"
```

如果命令执行中卡在下面信息（如提示有差异，请反馈给我）：

```bash
==> Tapping homebrew/core
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core'...
```

请`Control + C`中断脚本执行如下命令：

```bash
cd "$(brew --repo)/Library/Taps/"
mkdir homebrew && cd homebrew
git clone git://mirrors.ustc.edu.cn/homebrew-core.git
```

**`cask` 同样也有安装失败或者卡住的问题，解决方法也是一样：**

```bash
cd "$(brew --repo)/Library/Taps/"
cd homebrew
git clone https://mirrors.ustc.edu.cn/homebrew-cask.git
```

成功执行之后继续执行前文的安装命令:

```bash
/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"
```

最后看到`==> Installation successful!`就说明安装成功了。

最最后更新下：

```bash
brew update
```

1、安装 node
`brew install node`

2、npm升级
npm是随着nodejs安装一并安装的。 更新npm，可以用npm命令。
`npm install npm -g`

3、npm 常用命令
本地安装, 安装在当前目录
npm install xx

查看所有全局安装的模块
npm list -g

查看某个模块的版本号
npm list grunt

卸载模块
npm uninstall xxx
卸载后，可以cd到node_modules/目录下查看，或者使用命令查看：
npm ls

更新模块
npm update xx

搜索模块
nm search xx





1.安装cnpm

(1) 输入以下命令

| 1    | `npm install -g cnpm --registry=https:``//registry.npm.taobao.org` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

(2) 输入cnpm -v输入是否正常，这里肯定会出错。

| 1    | `cnpm -v` |
| ---- | --------- |
|      |           |

cnpm install --global vue-cli





# 问题 遇到[Error of “error: could not lock config file .git/config: Permission denied” occurs while installing Carthage](https://stackoverflow.com/questions/49282002/error-of-error-could-not-lock-config-file-git-config-permission-denied-occu)

Check for the permissions on these files.

```
ls -l /usr/local/Homebrew/.git/FETCH_HEAD
ls -l /usr/local/Homebrew/Library/Taps/caskroom/homebrew-cask/.git/FETCH_HEAD
ls -l /usr/local/Homebrew/Library/Taps/dart-lang/homebrew-dart/.git/FETCH_HEAD
ls -l /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/.git/FETCH_HEAD
```

If you don't have the permissions, run

```
sudo chown -R $(whoami):admin /usr/local/* && sudo chmod -R g+rwx /usr/local/*
```

In High Sierra, Run this command instead:

```
sudo chown -R $(whoami) $(brew --prefix)/*
```

You can also see the related github issues [here](https://github.com/Homebrew/legacy-homebrew/issues/43471)