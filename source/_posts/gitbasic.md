---
title: git常用命令
date: 2016-05-07 11:07:46
tags: git
---

## 概述

git 作为我们常用的版本管理工具，提供了丰富的命令操作,这里记录一下开发中我们常用的命令，以备不时之需。

<!-- more -->

## 配置

首先，安装完git后，我们需要用下面的命令来配置git。

```java
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

## 查看

- `pwd` :用于显示当前目录.
```java

BoBoMEedeMacBook-Pro:RxJavaLearn bobomee$ pwd
/Users/bobomee/Desktop/rx/RxJavaLearn
```

## 时光穿梭

- `git reflog`:查看命令历史,`commit id`，用于回到未来的个版本
```java
$ git reflog
ea34578 HEAD@{0}: reset: moving to HEAD^
3628164 HEAD@{1}: commit: append GPL
ea34578 HEAD@{2}: commit: add distributed
cb926e7 HEAD@{3}: commit (initial): wrote a readme file
```

## 撤销修改(`git status`查看)
- `git checkout -- file`:撤销file的修改到最近一次`add/commit`的状态
```java
$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
```
同时，如果误删文件，用此命令可以恢复

- `git reset HEAD file`:撤销暂存区的修改（unstage），重新放回工作区.其中`HEAD`，表示最新的版本
```java
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   readme.txt
#
```

## 远程仓库

本地与远程库使用ssh加密，需要配置`SSH Key`。

通过如下命令，查看是否存在`id_rsa.pub`
```java
$ ls -al ~/.ssh
```
如果没有的话，通过如下命令来生成

```java
$ ssh-keygen -t rsa -C "email@example.com"
```
如果有的话，则使用如下命令来查看

```java
$ cd ~/.ssh
$ cat id_rsa.pub
```
将SSH添加到远程库后，使用如下命令来测试是否可用

```java
$ ssh -T git@github.com
```

与远程仓库关联

```java
$ git remote add origin git@github.com:BoBoMEe/bobomee.github.io.git
```

移除远程仓库

```java
$ git remote rm origin
```

查看远程仓库

```java
$ git remote -v
```
推送到远程库

```java
$ git push origin master
```
第一次使用，使用参数`u`与远程仓库联系起来

```java
$ git push -u origin master
```

## 分支管理

常用命令如下：

```java
$ git branch // 查看分支

$ git branch <name> //创建分支

$ git checkout <name> //切换分支

$ git checkout -b <name> //创建+切换分支

$ git merge <name> //合并某分支到当前分支

$ git branch -d <name> //删除分支
$ git branch -D <name> //强行删除，适用尚未merge的分支

$ git log --graph  //看到分支合并图
```
### 分支管理策略

`--no-ff`模式，会在merge时生成一个新的commit，而`fast forward(Default)`j没有，
用`--no-ff`模式方便以后查看历史

```java
$ git merge --no-ff -m "merge with no-ff" dev
```

* 常用分支<br/>

`master`稳定分支，仅用来发布新版本，平时不在上面干活

`dev`开发分支，团队成员每个人都在自己到分支上工作，并merge到dev，开发完成合并到master

* 临时分支

bug（fixbug）分支,紧急修复bug用，最终合并到master和dev，并删除，不推送到远程

预发布（release）分支，合并到master之前的测试版本，最终合并到dev和master，并删除

功能（feature）分支，用于新功能开发，最终合并到dev，并删除

### 工作区保存

比如正在dev分支进行开发，需要新建bug分支来修复bug，开发尚未完成，不好提交
此时需要用到`stash`功能

```java
// 把当前工作现场“储藏”起来，等以后恢复现场后继续工作
$ git stash
Saved working directory and index state WIP on dev: 6224937 add merge
HEAD is now at 6224937 add merge
```

```java
// 查看stash 列表
$ git stash list
stash@{0}: WIP on dev: 6224937 add merge
```

```java
//恢复stash的，并删除stash内容
$ git stash pop
# On branch dev
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   hello.py
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   readme.txt
#
Dropped refs/stash@{0} (f624f8e5f082f2df2bed8a4e09c12fd2943bdd40)
```
使用`git stash apply`，stash内容并不删除，使用`git stash drop`来删除

```java
//恢复指定的stash
$ git stash apply stash@{0}
```

## 多人协作

### 贡献开源

github上众多的开源项目都是用git来进行管理的，我们可以通过fork/pull request
来贡献开源，标准的操作流程中，我们需要注意以下一些地方。

* 不要在master上进行更改，为我们的主题创建一个单独的branch。

```java
$ git checkout -b fixbug
```

* rebase与merge

比如源仓库有更新时，最好使用rebase，这样提交pullrequest的时候不会产生多余的commit

* 同步源仓库修改

如果上游仓库更新，这时候我们想要新的特性的时候，可以使用如下命令

```java
$ git branch --set-upstream-to=<repo_name>/master master
```

### 团队开发

如果是团队开发的话，一般都会在dev分支上干活，从远程库上clone下来后，在本地默认只会看到master分支

ps:命令查看
```java
$ git branch
* master
```

所以必须在本地创建和远程分支对应的分支dev，并和远程关联。

```java
$ git checkout -b dev origin/dev
```

在推送到远程库之前，pull的时候，如果出现`no tracking information`,
则可以使用如下命令,建立本地分支和远程分支的关联

```java
$ git branch --set-upstream-to=<repo_name>/master master
```

参考：

[廖雪峰的官方网站](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013760174128707b935b0be6fc4fc6ace66c4f15618f8d000)

[Pull Request的正确打开方式（如何在GitHub上贡献开源项目](http://blog.csdn.net/zhangdaiscott/article/details/17438153)
