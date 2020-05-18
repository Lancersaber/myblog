---
title: git clone,git pull,git fetch的区别
categories: git
---
# git clone,git pull,git fetch的区别
---
## git clone
git clone顾名思义就是将其他仓库克隆到本地，包括被clone仓库的版本变化。举个例子，你当前目录比方说是在e:/course/中，此时若想下载远程仓库，本地无需git init,直接git clone url（url是你远程仓库的地址，直接复制就可以了）。执行git clone等待clone结束，e:/course/目录下自动会有一个.git的隐藏文件夹（如果看不见，请尝试设置隐藏文件夹可见），因为是clone来的，所以.git文件夹里存放着与远程仓库一模一样的版本库记录。clone操作是一个从无到有的克隆操作，再次强调不需要git init初始化。

###git clone用法
```
$ git clone <版本库的url>
```

样就会在本地生成一个目录，该目录与远程仓库同名。
However，如果本地目录不想与远程仓库同名怎么办？？也有办法，将目录名作为git clone命令的第二个参数:
```
$ git clone <版本库的网址> <本地目录名>
```

## git pull
git pull是拉取远程分支更新到本地仓库的操作。比如远程仓库里的学习资料有了新内容，需要把新内容下载下来的时候，就可以使用git pull命令。事实上，git pull是相当于从远程仓库获取最新版本，然后再与本地分支merge（合并）。
　　即：git pull = git fetch + git merge
	注：git fetch不会进行合并，执行后需要手动执行git merge合并，而git pull拉取远程分之后直接与本地分支进行合并。更准确地说，git pull是使用给定的参数运行git fetch，并调用git merge将检索到的分支头合并到当前分支中。

### git pull用法
```
$ git pull <远程主机名> <远程分支名>:<本地分支名>
```
表示将远程主机的远程分支与本地的分支拉取下来并进行合并

相比起来，git fetch更安全也更符合实际要求，因为可以在merge前，我们可以查看更新情况，根据实际情况再决定是否合并。

## git fetch更新远程代码到本地仓库
理解 fetch 的关键, 是理解 FETCH_HEAD，FETCH_HEAD指的是: 某个branch在服务器上的最新状态’。这个列表保存在 .Git/FETCH_HEAD 文件中, 其中每一行对应于远程服务器的一个分支。
当前分支指向的FETCH_HEAD, 就是这个文件第一行对应的那个分支.
一般来说, 存在两种情况:
+ 如果没有显式的指定远程分支, 则远程分支的master将作为默认的FETCH_HEAD
+ 如果指定了远程分支, 就将这个远程分支作为FETCH_HEAD
```
方法一
$ git fetch origin master                #从远程的origin仓库的master分支下载代码到本地的origin master
$ git log -p master.. origin/master      #比较本地的仓库和远程参考的区别
$ git merge origin/master                #把远程下载下来的代码合并到本地仓库，远程的和本地的合并

方法二
$ git fetch origin master:temp           #从远程的origin仓库的master分支下载到本地并新建一个分支temp
$ git diff temp                          #比较master分支和temp分支的不同
$ git merge temp                         #合并temp分支到master分支
$ git branch -d temp                     #删除temp
```