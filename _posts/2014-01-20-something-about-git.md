---
layout: post
title: Git相关知识整理
category : learn
tags : [git]

---

刚开始用Git的时候是在差不多一年以前了。那时侯刚进公司，开始接触linux和android的东西，而公司里面是用Git来管理源码。
在没有用Git之前，可以说都没有管理代码的概念，只是用复制来简单备份一下而已，真是惭愧啊! 
虽然在那之前曾经折腾过SVN，但始终没用起来，对版本控制也没深入了解。

记录一下Git相关的一些知识。

<!--break-->

### Git介绍

#### Git是什么
维基百科是这样描述[Git](http://zh.wikipedia.org/wiki/Git)的：
>Git是一个分布式版本控制／软件配置管理软件，原来是linux内核开发者林纳斯·托瓦兹（Linus Torvalds）为了更好地管理linux内核开发而创立的。

>Git是用于Linux内核开发的版本控制工具。与CVS、Subversion一类的集中式版本控制工具不同，它采用了分布式版本库的作法，不需要服务器端软件，就可以运作
版本控制，使得源代码的发布和交流极其方便。Git的速度很快，这对于诸如Linux kernel这样的大项目来说自然很重要。Git最为出色的是它的合并追踪（merge  tracing）能力。


>Git和其他版本控制系统（如CVS）有不少的差别，Git本身关心档案的整体性是否有改变，但多数的CVS，或Subversion系统则在乎档案内容的差异。因此Git更像一
个档案系统，直接在本机上取得资料，不必连线到host端取资料回来。

#### Git相关术语

*  分支（Braches)  
一个分支意味着它是一个独立拥有自己历史版本信息的代码线。你可以从已有的代码中生成一个新的分支，这个分支与其余的分支完全独立。默认的分支叫做master。用户可以选择一个分支，选择一个分支叫做Checkout.

*  提交（Commit)  
当你提交你的更改到Git库中，它将创建一个新的提交对象。这个提交对象会有一个新版本的唯一标识。本次修订后，可以检索，例如，如果你想看到一个旧版本的源代码。每个提交对象中都会包含修改者和提交者，从而有可以确定是谁做了改变。修改者和提交者，可以是不同的人。

*  仓库（Repository)  
仓库包含了随着时间的推移和各种不同的分支和标签不同版本历史。在Git仓库的每个副本是一个完整的信息库。你可以从仓库中获取你的工作副本。

*  标记（Tags）  
标记指的是某个分支某个特定时间点的状态。通过标记，可以很方便的切换到标记时的状态。

*  工作树/区（Working tree)   
工作区中包含了仓库的工作文件。您可以修改的内容和提交更改作为新的提交到仓库。

*  暂存区（Staging area）  
暂存区是工作区用来提交更改（commit）前可以暂存工作区的变化。暂存区包含了工作区的一系列更改快照，这些快照可以用来创建新的提交。 

#### Git其它推荐
**Git相关代码托管服务：**


* [GitHub](http://www.github.com/): 大名鼎鼎的国外代码托管服务。

* [git@osc](http://git.oschina.net/): 开源中国的代码托管服务。

当然还有其它的，但是我一般都是用上面这两个。

**Git的一些比较好的书籍：**

* [Pro Git](http://git.oschina.net/progit/): Git社区书，经典。

* [Git权威指南](http://www.worldhello.net/gotgit/): 国人写的很好的一本关于Git的书。

* [GotGitHub](http://www.worldhello.net/gotgithub/): Git权威指南作者写的一本关于GitHub的开源电子书。

### Git常用命令备忘

#### Git配置

```bash
git config --global user.name "xxx" 			# 设置用户名，会在log里面显示
git config --global user.email "xxx@gmail.com"	# 设置用户邮箱，会在log里面显示
git config --global color.ui true 				# 配置终端输出着色支持
git config --global core.editor vi 		   		# 设置Editor使用vi
git config -l   								# 列举所有配置

用户的git配置文件~/.gitconfig
```

#### Git常用命令

**建库、查看、添加、提交、删除、找回，重置修改文件**

```bash
git help <command>   			# 显示command的help

git init						# 建立Git版本库
git init --bare					# 建立Git裸版本库

git show 						# 显示最新一次提交修改
git show $id         			# 显示某次提交的修改记录

git chcekout  <file>   			# 抛弃工作区文件<file>的修改
git checkout  .        			# 抛弃工作区修改

git add <file>      			# 将工作文件修改提交到本地暂存区
git add .           			# 将所有修改过的工作文件提交暂存区

git rm <file>      	 			# 从版本库中删除文件
git rm <file> --cached  		# 从版本库中删除文件，但不删除文件

git reset <file>    			# 从暂存区恢复到工作文件
git reset -- .      			# 从暂存区恢复到工作文件
git reset --hard    			# 恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改

git commit						# 本地提交
git commit -a       			# 将git add, git rm和git commit等操作都合并在一起做
git commit -am "some comments" 	# 把所有修改以some comments为log来提交
git commit --amend      		# 修改最后一次提交记录

git revert <$id>    			# 恢复某次提交的状态，恢复动作本身也创建了一次提交对象
git revert HEAD     			# 恢复最后一次提交的状态
```

**查看文件diff**

```bash
git diff 		     			# 比较当前文件和暂存区文件差异
git diff <file>					# 比较当前文件<file>和暂存区文件差异
git diff <$id1> <$id2>   		# 比较两次提交之间的差异
git diff <branch1> <branch2> 	# 在两个分支之间比较 
git diff --staged   			# 比较暂存区和版本库差异
git diff --cached   			# 比较暂存区和版本库差异
git diff --stat     			# 仅仅比较统计信息
```

**查看提交记录**

```bash
git log				# 查看全部提交记录
git log <file>      # 查看该文件每次提交记录
git log -p <file>   # 查看每次详细修改内容的diff
git log -p -2       # 查看最近两次详细修改内容的diff
git log --stat      # 查看提交统计信息
```

#### Git 本地分支管理

**查看、切换、创建和删除分支**

```bash
git branch -a			# 查看本地与远程的全部分支
git branch -r           # 查看远程分支
git branch <new_branch> # 创建新的分支
git branch -v           # 查看各个分支最后提交信息
git branch --merged     # 查看已经被合并到当前分支的分支
git branch --no-merged  # 查看尚未被合并到当前分支的分支

git checkout <branch>     # 切换到某个分支
git checkout -b <new_branch> # 基于当前分支创建新的分支，并且切换过去
git checkout -b <new_branch> <branch>  # 基于branch创建新的new_branch

git checkout $id          # 把某次历史提交记录checkout出来，但无分支信息，切换到其他分支会自动删除
git checkout $id -b <new_branch>  # 把某次历史提交记录checkout出来，创建成一个分支

git branch -d <branch>  # 删除某个分支
git branch -D <branch>  # 强制删除某个分支 (未被合并的分支被删除的时候需要强制)
```

**分支合并和rebase**

```bash
git merge <branch>               # 将branch分支合并到当前分支
git merge origin/master --no-ff  # 不要Fast-Foward合并，这样可以生成merge提交

git rebase master <branch>       # 将master rebase到branch
```

#### Git补丁管理

```bash
git diff > ../sync.patch         # 生成补丁
git apply ../sync.patch          # 打补丁
git apply --check ../sync.patch  # 测试补丁能否成功
```

#### Git暂存管理

```bash
git stash                        # 暂存
git stash list                   # 列所有stash
git stash apply                  # 恢复暂存的内容
git stash drop                   # 删除暂存区
```

#### Git远程分支管理

```bash
git pull                         # 抓取远程仓库所有分支更新并合并到本地
git pull --no-ff                 # 抓取远程仓库所有分支更新并合并到本地，不要快进合并
git fetch origin                 # 抓取远程仓库更新
git merge origin/master          # 将远程主分支合并到本地当前分支
git checkout --track origin/branch     # 跟踪某个远程分支创建相应的本地分支
git checkout -b <loca_branch> origin/<remote_branch>  # 基于远程分支创建本地分支，功能同上
git branch --set-upstream develop origin/develop 	# 设置跟踪远程库分支和本地库分支

git push                         # push所有分支
git push origin master           # 将本地主分支推到远程主分支
git push -u origin master        # 将本地主分支推到远程(如无远程主分支则创建，用于初始化远程仓库)
git push origin <branch>   		 # 将本地分支推到远程分支
git push origin <local_branch>:<remote_branch>  # 提交本地local分支作为远程的remote分支
git push origin --delete <branchName> # 删除远程分支
```

#### Git远程仓库管理

```bash
git remote -v                    # 查看远程服务器地址和仓库名称
git remote show origin           # 查看远程服务器仓库状态
git remote add origin git@github:robbin/robbin_site.git         # 添加远程仓库地址
git remote set-url origin git@github.com:robbin/robbin_site.git # 设置修改远程仓库地址
git remote set-head origin master   # 设置远程仓库的HEAD指向master分支
git remote rm <repository>       # 删除远程仓库
```

