---
title: Git实用命令
categories:
- 工具
tags:
- Git
---

- 本地仓库操作
- 远程仓库操作
- 分支管理
- 标签管理
- 其他

<!--more-->


# 本地仓库操作
============================本地仓库操作============================
创建版本库
git init

查看工作区、暂存区、本地版本库之间的状态
git status

对比暂工作区、暂存区，可指定文件
git diff
git diff <file>

添加文件到暂存区
git add .
git add <file>...

提交文件到本地版本库
git commit -m "remark message"

本地版本库操作日志，至当前版本
git log

本地版本库操作日志，所有操作
git reflog

版本回退，回退指定版本并更新工作区
git reset --hard HEAD^
git reset --hard HEAD^^
git reset --hard HEAD~100
git reset --hard <commit id>

撤销修改，还原工作区到暂存区状态，暂存区干净时则到版本库状态
git checkout -- <file>...

撤销暂存区的修改
git reset HEAD
git reset HEAD <file>

删除文件，以下只提交到暂存区
git rm <file>


# 远程仓库操作
============================远程仓库操作============================
为本地仓库增加远程服务器端
git remote add origin git://github.com/someone/another_project.git
git remote add origin git@server-name:path/repo-name.git

推送本地分支内容到远程仓库（由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。）
git push -u origin master
git push origin master

克隆远程仓库到本地
git clone git@github.com:someone/another_project.git

修改origin指定地址
git remote set-url origin 新地址

# 分支管理
==============================分支管理==============================
创建分支
git branch dev
切换分支
git checkout dev
创建并切换分支
git checkout -b dev
查看当前分支
git branch
删除分支
git branch -d dev
强行删除未合并的分支，直接丢弃分支
git branch -D <name>

合并分支，指定目标分支合并到当前分支
git merge dev
不使用fast forward合并分支，为保留历史分支信息
git merge --no-ff -m "merge with no-ff" dev

查看分支合并记录
git log --graph --pretty=oneline --abbrev-commit

保存并隐藏现有工作区及暂存区内容
git stash
查看已保存隐藏的工作区及暂存区内容列表
git stash list
恢复已保存隐藏的工作区及暂存区内容，未删除列表
git stash apply <stash@{0}>
删除已保存隐藏的工作区及暂存区内容列表
git stash drop <stash@{0}>
恢复已保存隐藏的工作区及暂存区内容，并删除列表中数据
git stash pop

查看远程库信息
git remote
git remote -v

推送分支内容
git push origin branch-name

建立本地分支和远程分支的关联
git branch --set-upstream dev origin/<branch>

更新分支内容
git pull
git pull <remote> <branch>

本地创建和远程对应的分支
git checkout -b branch-name origin/branch-name


# 标签管理
==============================标签管理==============================
创建标签
git tag <tagname>
git tag <tagname> <commit id>

创建标签，用-a指定标签名，-m指定说明文字
git tag -a <tagname> -m "version 0.1 released" <commit id>

创建标签，通过-s用私钥签名
git tag -s <tagname> -m "signed version 0.2 released" <commit id>

查看所有标签
git tag

查看标签信息
git show <tagname>

删除标签
git tag -d <tagname>

推送标签
git push origin <tagname>

推送所有标签
git push origin --tags

删除远程标签
git push origin :refs/tags/<tagname>


# 其他
================================其他=================================
让Git显示颜色
git config --global color.ui true

忽略跟踪文件.gitignore

设置命令缩写
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.unstage 'reset HEAD'
git config --global alias.last 'log -1'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

配置文件位置.git/config



多人协作步骤：
1、尝试用git push origin branch-name推送分支；
2、如果推送失败，则需更新远程分支至本地合并，用git pull试图合并；
3、如果合并有冲突，则解决冲突，并在本地提交；
4、再次用git push origin branch-name推送分支。
