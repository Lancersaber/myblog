---
title: git的常用指令
categories: git
---
# git的常用指令
---
## 关于远程仓库操作的指令
### 克隆整个仓库
```
$ git clone [url]
```
### 查看远程仓库
```
$ git remote -v
```
### 添加远程仓库
```
$ git remote add [name] [url]
```
### 删除远程仓库
```
$ git remote rm [name]
```
### 修改远程仓库
```
$ git remote set-url --push [name] [newUrl]
```
### 拉取远程仓库
```
$ git pull [remoteName] [localBranchName]
```
### 推送远程仓库
```
$ git push [remoteName] [localBranchName]
```
如果想把本地的某个分支test提交到远程仓库，并作为远程仓库的master分支，或者作为另外一个名叫test的分支，如下：
```
$git push origin test:master // 提交本地test分支作为远程的master分支
$git push origin test:test // 提交本地test分支作为远程的test分支
```

## 关于分支的操作
### 查看本地分支
```
$ git branch
```
### 查看远程分支
```
$ git branch -r
```
### 创建本地分支
```
$ git branch [name] ----注意新分支创建后不会自动切换为当前分支
```
### 切换分支
```
$ git checkout [name]
```
### 创建新分支并立即切换到新分支
```
$ git checkout -b [name]
```
### 删除分支
```
$ git branch -d [name] ----d选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用-D选项
```
### 合并分支
```
$ git merge [name] ----将名称为[name]的分支与当前分支合并
```
### 创建远程分支(本地分支push到远程)
```
$ git push origin [name]
```
### 删除远程分支
```
$ git push origin :heads/[name] 或 $ git push origin :[name]
```
### 创建空的分支：(执行命令之前记得先提交你当前分支的修改，否则会被强制删干净没得后悔)
```
$git symbolic-ref HEAD refs/heads/[name]
$rm .git/index
$git clean -fdx
```

## 附 Git常用命令速查
### 查看当前状态
```
git status
```
### 查看所有的分支
```
git branch -a 
```
### 显示远程库origin里的资源
```
git remote show origin 
```
### 查看远程库
```
git remote show
```
### 删除一个文件
```
git rm [file name] 
```
