---
title: Git常用命令
---

工作中常用到的git使用命令和技巧
<!-- more -->

## 基本命令

git add .   添加全部到暂存区
git commit -m "备注信息"    提交暂存的修改
git commit --am     合并提交
git pull    拉取代码并合入 === git fetch && git merge
git fetch   只拉取，不合入
git checkout .  放弃所有没加入暂存区的代码

github上创建仓库
git remote add origin https://github.com/blue-squirrel/仓库名.git   设置远程仓库
git push -u origin master   推送到远程分支

## 分支

git branch  列出本地分支
git branch -a   列出本地和远程分支
git branch -D 分支名    强制删除分支
git checkout -b 分支名 origin/分支名    拉取远程到本地

## git stash 暂存

git stash   把本地的改动暂存
git stash save "message"    执行存储时，添加备注
git stash pop   应用最近一次暂存，并删除暂存记录
git stash apply     应用某个存储,但不会把删除，默认使用第一个,即 stash@{0}，如果要使用其他个，git stash apply stash@{$num} 。
git stash list     查看stash列表
git stash clear     删除所有stash

## 合并前几次commit为一次

![avatar](http://mms2.baidu.com/it/u=2534115628,2023806309&fm=253&app=138&f=PNG&fmt=auto&q=75?w=417&h=203)

git rebase -i <base-commit>

以base-commit为基准，合并基准之后的所有commit

将类似

pick  第一次提交
pick  第二次提交
pick  第三次提交
pick  第四次提交

改成

pick  第一次提交
s     第二次提交
s     第三次提交
s     第四次提交

然后会弹出提交commit注释的编辑，保存后会保留注释并合并成一条

## git rebase

![avatar](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec7db69f93ee440a8a5b9b62dd68668c~tplv-k3u1fbpfcp-watermark.awebp)

### 拉取远程的新代码

git pull --rebase

rebase 会将两个分支进行合并，同时合并之前的 commit 历史。如果出现冲突，解决冲突后执行以下命令即可：

git add
git rebase --continue   继续变基

### 公共代码库提交PR，拉取新代码

git pull --rebase upstream master

修改后 git add . && git commit -m ""

git push -f 即可
