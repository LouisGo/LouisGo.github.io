---
title: 常用git指令和场景
date: 2018-04-24 17:15:16
tags: 
- git
categories: 
- 前端
keywords: git,git指令,git教程
---
<div class="note info"><p>下面列举一些开发过程中自己遇到的指令，后续会不断完善和修改。</p></div>

# Local操作

## 全局变量

设置用户名

```
$ git config --global user.name "<yourname>"
```

设置邮箱

```
$ git config --global user.email "<youremail>"
```

设置颜色提醒

```
$ git config --global color.ui "always"
```

## 初始化新版本库

```
$ git init
```

## 添加新文件到版本库

添加单个文件

```
$ git add somefile.txt
```

添加所有txt文件

```
$ git add *.txt
```

添加所有文件（包括子目录，不包括空目录）

```
$ git add .
```

<!--more-->

## 提交

提交已添加的文件并添加注释

```
$ git commit -m "some commit text"
```

提交已添加的单个文件并添加注释

```
$ git commit -m "some commit text" readme.md
```

提交所有修改（自动git add .）

```
$ git commit -m "some commit text" -a
```

不会产生新的提交历史记录

```
$ git commit -C head -a --amend
```

## 未提交时撤销修改

将工作区未add的单个文件撤回

```
$ git checkout head readme.md
```

将工作区未add的所有文件撤回

```
$ git checkout head .
```

将暂存区的单个文件撤回到工作区

```
$ git reset head readme.md
```

将暂存区的所有文件撤回到工作区

```
$ git reset head
```

复位到head之前的那个版本

```
$ git reset --hard head 
```

## 已提交时撤销修改

获取commit_id
```
$ git log
```

- 完成撤销，同时将代码恢复到前一commit_id 对应的版本，已修改的代码被覆盖

```
$ git reset --hard commit_id
```

- 完成撤销, 但是不对代码修改进行撤销，已修改的代码仍在工作区，等待进一步操作

```
$ git reset commit_id
```

## 分支

列出本地分支

```
$ git branch
```

列出所有分支

```
$ git branch -a
```

基于当前分支末梢创建新分支

```
$ git branch <branchname>
```

检出/切换指定分支

```
$ git checkout <branchname>
```

创建并检出（上面两步的语法糖）

```
$ git checkout -b <branchname>
```

合并并提交

```
$ git merge <branchname>
```

重命名分支，不覆盖已存在的同名分支

```
$ git branch -m <branchname> <newnbranchname>
```

重命名分支，覆盖已存在的同名分支

```
$ git branch -M <branchname> <newnbranchname>
```

删除分支，如果没有被合并会删除失败

```
$ git branch -d <branchname>
```

删除分支

```
$ git branch -D <branchname>
```

## 状态

查看状态

```
$ git status
```

查看日志

```
$ git log
```

查看当前分支历史记录

```
$ gitk
```

查看指定分支历史目录

```
$ gitk <branchname>
```

查看所有分支历史目录

```
$ gitk gitk --all
```
# Remote

## 克隆版本库

克隆后会自动添加4个config

```
$ git clone <url>
```

## 分支

列出远处分支

```
$ git branch -r
```

删除远程库中已经不存在的分支

```
$ git remote purne origin
```

## 查看

查看当前的远程库

```
$ git remote
$ git remote -v
```

查看远程仓库信息

```
git remote show <remotename>
```

## 获取

获取但不合并

```
$ git fetch <remotename>
```

获取并合并到当前分支

```
$ git pull 
```

## 推送

如果已经设置了推送地址

```
$ git push
```

否则需要先设置

```
$ git push --set-upstream <remotename><branchname>
```

## SSH key

1. 启动Git Bash控制台

2. 如果之前生成过，需要先备份
```
$ cd ~/.ssh
$ mkdir key_backup
$ cp id_rsa*key_backup
```

3. 生成SSH Key
```
$ ssh-keygen -t rsa -C "<youremail>"
```

4. 将id_rsa.pub的内容粘贴到github中