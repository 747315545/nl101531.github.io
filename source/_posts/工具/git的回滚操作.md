---
title: git的回滚操作
subtitle: 公司发布系统出现问题,利用git reset进行强制回滚操作.
cover: http://oobu4m7ko.bkt.clouddn.com/1514805018.png
author: 
  nick: 屈定
tags:
  - git    
categories: git
date: 2017-11-20 22:39:59
updated: 2017-11-20 22:40:01
---
一直在使用git作为版本管理器,但是有一次线上出问题,但是又有未通过完整测试的代码在mater分支上,那么所需要的操作就是 `回滚代码到上一个已通过测试的master版本 ` -> `修复bug` -> `发布` -> `还原master` -> `合并修改了bug的分支 ` -> `重新上预发布`,那么下面开始演练.

为了简单描述,我们把定义几个分支.
init-master //最初的master
untest-master  //未完全通过测试的master,其是远程仓库上最新的master
bugfix/fix_some_bug  //修改的bug分支

### 操作

**拉取untest-master**
首先是切换并更新master,注意此时的master是`untest-master`,也就是我们要对其进行回滚操作
```sh
git checkout master
git pull origin master
```
**回滚到init-master**
回滚操作首先需要定义到要回滚到的提交记录,然后强制回滚到该记录上,注意这里的操作只是对本地分支的操作并不会影响到远程分支 
```sh
//使用git log查看其提交记录,确定要回滚到的`commit id`
git log 
g reset --hard [commit id]
```
**回滚远程master**
回滚操作是把你当前的分支强制提交到master上也就是加`-f`参数指明**强制覆盖**,该命令需要你有相应的master权限.这样的话master上就是之前的版本了.那么接下来就是一般的修改bug了.
```sh
git push -u origin master -f
```
**修复bug,并发布**
bug的修复是在当前`init-master`分支的基础上,切出一个新的分支,然后像平常一样修改提交,最后合到`init-master`上.
```sh
git checkout -b bugfix/fix_some_bug
// 修复bug
// 发布
```
**取消回滚**
取消回滚则是重新强制恢复到修改过的版本,然后就可以强制回滚到任意版本了
```sh
git reflog  查看该分支的变动,可以确定其commit id
git reset --hard [commit id]
```
**合并修改了bug的分支**
这次回滚后你处在`untest-master`分支上,该分支上是没有`bugfix/fix_some_bug`修复bug的代码的,那么把他合并过来.然后推送上去,那么master就是当前最新的版本了
```sh
git merge bugfix/fix_some_bug
git push origin master
```

### 总结
以上是操作是必须需要有master权限的,否则无法回滚,另外如果担心代码丢失那么在`untest-master`分支上再次checkout一个分支,这样即使master再怎么变,这个分支仍然是`untest-master`的副本.
最后git是一款强大的版本工具,其可能有更加简单的方法,欢迎分享.
