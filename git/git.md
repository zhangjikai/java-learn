# Git
<!-- toc -->

## Gti文件声明周期

![](/git/images/lifecycle.png)

git目录中的文件主要有两种状态：
1. tracked（被追踪），指已经被纳入版本控制的文件，在上一次的快照中（snapshot）有它们的记录。它们的状态可能是unmodified,modified或者staged
2. untracked（未追踪），指即没有存在于上一次的快照中，也没有放到暂存区（staging area）。比如新创建的问题件。


## commit

### amend
重置上一次的amend
```
git reset --soft HEAD@{1}
```

## 常用操作

### 查看一个文件的更改记录及每次更改内容
```
gitk filename
```
