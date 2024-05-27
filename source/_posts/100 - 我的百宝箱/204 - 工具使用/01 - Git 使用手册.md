---
title: Git 使用手册
date: 2024-01-02
categories:
  - 工具使用
tags:
  - Git
published: true
---
# 0 参考资料

1. [一文让你了解如何为 Git 设置代理](https://ericclose.github.io/git-proxy-config.html)

# 1 Git 基本使用

## 1.1 配置 Git
使用Git的第一件事就是设置你的名字和email,这些就是你在提交commit时的签名，每次提交记录里都会包含这些信息。使用git config命令进行配置：

```bash
git config --global user.name “BaihlUp”
git config --global user.email “2677443264.com”
git config --global credential.helper store #保存登陆信息，以后不用每次都输入密码
cat ~/.git-credentials #登陆票证信息
git config --global push.default "current" #设置每次默认push的分支为当前分支

如果使用ssh则：ssh-keygen -t rsa -C "邮箱"
[root@baihl testWaf]# git push origin dev
Username for 'https://github.com': BaihlUp
Password for 'https://BaihlUp@github.com': 填Github Token
```

## 1.2 设置代理

- 全局代理

```bash
git config --global http.proxy <protocol>://<host>:<port>
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

- 为指定域名设置代理
```bash
git config --global http.<url>.proxy <protocol>://<host>:<port>
#为指定域名gitHub.com设置代理
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

# 2 Git 常用操作

## 2.1 Git 常用命令

```bash
git clone url ：复制仓库
git diff ：如果修改的文件没有加入缓存区，则显示出修改文件和缓存区的差异
git add file1 file2 file3 ：把文件加入到缓存区
git status
git diff --cached ：查看缓存区和本地仓库的差异
git commit -m “提交记录” ：提交缓存区的内容到本地仓库
git commit -a -m "add 3 files" ：直接把修改添加到缓存区并且提交
git rm ：删除文件后自动将已删除文件添加到缓存区
git init ：初始化本地仓库
git remote add origin 远程仓库地址
git push -u origin master 推送本地仓库到远程仓库
git remote show origin 显示远程仓库信息
git remote -v
```


## 2.2 Git 分支操作

```bash
git branch 分支名 ：创建分支
git branch 分支1 分支2 ：基于分支2创建一个分支1
git checkout -b 分支名 创建分支并切换到分支
git merge -m "合并记录" 分支名 合并分支
git branch -d 分支名 删除指定分支
git branch -d只能删除那些已经被当前分支的合并的分支. 如果你要强制删除某个分支的话就用
git branch –D 强制删除某个分支
git push origin --delete Chapater6 可以删除远程分支Chapater6
git reset --hard HEAD 回到合并之前的状态
git reset --hard ORIG_HEAD 撤销已经提交的合并
git branch -f 分支 HEAD~1 把分支强制移动到HEAD的父节点
```
## 2.3 Git日志

```bash
git log 查看提交记录
git log --stat 详细的提交记录
不同的显示log的方式：
git log --pretty=oneline
git log --pretty=short
git log --graph --pretty=oneline 显示提交线路图
git log --reverse 逆向显示所有log
git log --pretty=format:'%h : %s' --topo-order --graph 按拓扑顺序显示（子提交在父提交前）
git reset：恢复版本
git diff 分支1 分支2 ：比较两个分支的差异
git diff 分支名 ：查看当前分支和指定分支的差异
git diff 分支名 文件名 ：查看当前分支指定文件和指定分支的差异
git diff 分支名 --stat ：统计两个分支那些文件被改动
```

## 2.4 其他操作

- Git分布式工作流程（Git公共仓库）

如果多人在同一台电脑上工作，仓库未与远程仓库关联，当前工作项目为/home/shiyanlou/gitproject :
克隆本地仓库：

```bash
cd /tmp/
git clone /home/shiyanlou/gitproject myrepo 把本机的一个已有git仓库克隆过来
如果在myrepo中做修改，并提交到本地仓库，如果把myrepo的修改同步到gitproject呢？
cd /home/shiyanlou/gitproject
git pull /tmp/myhrepo master
```


如果经常修改远程分支，可以定义缩写：

```bash
git remote add myrepo /tmp/myrepo
git config --get remote.origin.url 查看本地仓库的远程仓库地址
```


Git标签

```bash
git tag 标签名 hash值
git tag -a stable-1 8c315325 -m "stable-1"
```


- git 根据tag创建分支

```bash
git branch <new-branch-name> <tag-name> 会根据tag创建新的分支
```


- Git忽略到某些文件

在项目的根目录下创建.gitignore文件，文件中以`#`开始的行为注释
```bash
foo.txt ：忽略掉文件名为foo.txt的文件
*.html ：忽略掉所有生成的html文件
!foo.html ：foo.html例外
*.[oa] ：忽略掉所有的.o 和 .a文件
```


- Git rebase操作
例如现在有个分支mywork，如果想让mywork分支合入远程origin，但是不带commit历史信息，可以使用git rebase操作如下：

```bash
git checkout mywork
git rebase origin
```

这个命令会把mywork分支里的每个提交取消掉，并且把他们临时保存为补丁（这些补丁存放在.git/rebase目录中），然后把mywork分支更新到最新的origin分支，最后把保存的这些补丁应用到mywork分支上。
在rebase的过程中可能出现冲突，需要解决冲突，然后用git add命令去更新这些内容的索引，然后执行
git rebase --continue继续应用余下的补丁。如果想终止rebase的行动，可以执行git rebase --abort

Git修复操作

```bash
git reset --hard HEAD^ 清空所有未提交的内容，不包括untracked files（未加入版本控制的文件）
git checkout -- file 把file从HEAD中签出，并且把它恢复成未修改时的样子
git revert HEAD 撤销最近的一个提交
git commit --amend 修改最近一次提交的log信息
```


维护GIT

```bash
git gc 压缩历史信息节约磁盘和内存空间
git fsck 运行一些仓库的一致性检查
```


找回丢失的对象

```bash
git stash save "message" 保存当前仓库未提交的改动
git stash list 查看刚才保存的
git stash clear 清空stash
git fsck --lost-found 找回刚才的删除
git show 哈希值 ：指定找回的哈希值，查看对应的内容
git merge 哈希值 ：被找回的内容合并到当前分支
```



```bash
git branch -D cool_branch 删除分支
git fsck --lost-found 找回刚才删除分支里面的提交对象
git show 哈希值 ： 查看一个找到的对象的内容
git rebase 哈希值 ：把对象的内容合并到当前分支

git reset --hard HEAD^ ：恢复到上上次提交的状态
git fsck --lost-found 把刚才的删除提交给找回来
git merge 哈希值 ：把删除提价合并
```


创建子模块
在主仓下创建一个仓库，在主仓下执行：

```bash
git submodule add 子仓库路径 子仓库名
```

子模块会加入主仓下gitmodules文件中


在主仓下更新子仓库

```bash
git submodule init
git submodule update
```


```bash
git whatchanged 每次修改的文件列表
git whatchanged --stat 每次修改的文件列表, 及文件修改的统计
git show 显示最后一次的文件改变的具体内容
git show -5 显示最后 5 次的文件改变的具体内容
git show commitid 显示某个 commitid 改变的具体内容
```

