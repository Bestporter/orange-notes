git

错误信息：

```
error: You have not concluded your merge (MERGE_HEAD exists).
hint: Please, commit your changes before merging.
fatal: Exiting because of unfinished merge.
```

解决：

```console
Git fetch —all   拉下所有的代码
Git reset —hard origin/master   重新取master分支的代码
```

command + e 显示文件列表，


# git push的时候报403的错误

修改.git目录下的config文件 URL这一行。

url = https://GitHub名字:GitHub密码@github.com/Daemongy/NanoDiaryAndroid.git
之后再push就行了


# git pull 报port 443错误
```console
git config --list
# 查看git config信息，git proxy信息
# 设置git config proxy
git config --global http.proxy http://F7688906:LPnXfx61@10.191.131.15:3128
# 格式，协议 用户名: 密码@地址:端口
# [protocol://][user[:password]@]proxyhost[:port]
```
