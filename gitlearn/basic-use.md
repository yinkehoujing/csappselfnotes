# 计算机教育缺失的一刻 第6讲

## 概念模型
1. tree/folder array of bolbs/files
2. bolbs (文件)
3. commit : 由许多结构体组成
 1. 父提交信息 (在分支中可能有多个父提交信息）
 2. author
 3. message
 4. snapshot: 记录当前提交的信息,作为历史信息图的一个节点
4. 所有commit信息实际上只存储对应信息的hash值 即reference to the object

## 本地操作
1. git help <command> git help add
2. git status 当前提交或工作目录的详细信息
3. git commit 提交暂存区的文件/文件夹 到 git仓库
4. git log 显示提交历史
   git log --all --graph --decorate
5. git checkout <hash-value> / <reference>
   git checkout master
   ==该操作将修改HEAD指针指向，默认HEAD指针指向上一次提交的快照状态==
6. git branch 显示所有分支
   git branch dog 新建一个分支
   (HEAD -> master, dog) 表示HAED指针指向master分支，同时master分支与dog分支指向   同一个节点状态,当此时提交一些东西 master指针便会移动到新的节点状态
7. git merge dog 合并到主分支(可能会出现合并冲突）
8. git diff 忘了（）
## 远程仓库
1. git remote 显示该本地仓库已知的所有远程仓库
2. git remote add <repo-name> <url> 添加远程仓库 一般repo-name取origin
3. git push <remote> <local branch>:<remote branch>
   向远程仓库提交本地仓库做的更改
4. 此时git log 会同时显示本地提交情况 和 远程提交情况
