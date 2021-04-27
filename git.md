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