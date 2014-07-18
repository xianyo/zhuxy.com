---
layout: post
title: Gitolite安装与配置
category: learn
tags: [git]

---

Gitolite 是一款 Perl 语言开发的 Git 服务管理工具，通过公钥对用户进行认证，并能够通过配置文件对写操作进行基于分支和路径的的精细授权。Gitolite 采用的是 SSH 协议并且使用 SSH 公钥认证，因此需要您对 SSH 了解，无论是管理员还是普通用户。因此在开始之前，请确认已经了解SSH相关知识。

<!--break-->

### 创建git用户
Ubuntu命令： 
 
```bash
$ sudo adduser --system --shell /bin/bash --group git
```  

CentOS命令：
  
```bash
# useradd --system --shell /bin/bash --create-home --home-dir /home/git git  
```

### 生成SSH key  

切换到git用户: 
 
```bash
$ su git
$ ssh-keygen
$ cp ~/.ssh/id_rsa.pub ~/git-admin.pub
```

### 下载安装gitolite  

```bash
$ git clone git://github.com/sitaramc/gitolite
```  
  
我是安装在git用户根目录下的。  
在根目录下创建bin文件夹  
然后执行：  

```bash
$ mkdir bin
$ ~/gitolite/install -to ~/bin
$ ~/bin/gitolite setup -pk ~/git-admin.pub
```  

成功后出现：  
初始化空的 Git 版本库于 /home/git/repositories/gitolite-admin.git/  
初始化空的 Git 版本库于 /home/git/repositories/testing.git/  
安装成功，更新就是重新安装。

### 测试  

还是在git用户下  

```bash
$ ssh git@127.0.0.1
```

如果返回类似这样的信息： 
 
```bash
hello git, this is git@linux-dev running gitolite3 v3.5.2-4-g62fb317 on git1.8.1.2
 R W    gitolite-admin
 R W    testing
```

代表gitolite工作正常  

### 配置
#### 添加用户
成功安装后gitolite会自动生成两个仓储，一个是`testing.git`用来测试，另一个`gitolite-admin.git`就是用来管理gitolite的配置仓储。
将`gitolite-admin.git` clone到本地，注意：还是在git用户下，因为当前只有git用户对其有读写权限。
  
```bash
$ git clone git@127.0.0.1:gitolite-admin.git
```  
 
成功clone到本地后，可以看到这个目录结构如下： 
 
```
├── conf   
│   └── gitolite.conf   
└── keydir   
    └── git-admin.pub   
```

conf是放置配置文件的目录，`gitolite.conf`就是gitolite的配置文件，包含对用户、仓储、仓储权限的配置。`keydir`目录用来放置所有的用户公钥。`git-admin.pub`为安装时`setup -pk`的那个用户公钥。  
 
添加用户，首先就是要把目标用户的公钥添加到`keydir`下，并重命名为该用户的`用户名.pub`，比如`dev1.pub`。
然后添加用户相应权限。  
只要是在`keydir`下存在的用户，都属于@all用户组，其他用户组可通过在`gitolite.conf`自行定义。  
如：  


```
@admin = git-admin dev1
 
repo gitolite-admin
    RW +       =    @admin
 
repo testing
    RW +       =   git-admin
    RW       =    @all
```   


@admin用户组有两个用户git-admin dev1，分别对应keydir下的git-admin.pub, dev1.pub。
gitolite-admin仓储的读/写/强制更新权限 只有@admin用户组拥有；
testing仓储的读/写/强制更新只有git-admin用户拥有，其他所有在keydir下存在公钥的用户享有读/写权限。
#### 权限配置

权限配置在gitolite.conf中进行，注释用#表示。

* C

	C 代表创建。仅在 通配符版本库 授权时可以使用。用于指定谁可以创建和通配符匹配的版本库。

* R, RW, 和 RW+

	R 为只读。RW 为读写权限。RW+ 含义为除了具有读写外，还可以对 rewind 的提交强制 PUSH。

* RWC, RW+C

	只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等）。

* RWD, RW+D

	只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。

* RWCD, RW+CD

	只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等），D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。

 
接下来实际分析一个稍微复杂一些的配置文件

```
1   @admin = git keven admin1 admin2
2   @devteam = dev1 dev2 dev3 fish
3 
4   repo gitolite-admin
5       RW+                 = git keven
6 
7   repo Projects/.+$
8       C                   = @admin
9       RW                  = @all
10 
11  repo testing
12      RW+                  =   @admin
13      -                    =   fish
14      RW      master       =   @dev
15      RW+     dev          =   dev1
16      RW      wip$         =   dev2
```

逐行解释：
1: @admin用户组有git keven admin1 admin2四个用户
2：@devteam用户组有dev1 dev2 dev3 fish四个用户
4：对于gitolite-admin仓储
5：git keven两个用户拥有读/写/强制更新的权限
7：对于Projects下所有的git仓储（/.+代表递归所有）
8：@admin用户组拥有创建仓储的权限
9：所有人均可读/写
11：对于testing.git
12：@admin用户组拥有读/写/强制更新的权限
13：fish是新手，对其屏蔽写的权限。因为其属@dev组，则还只剩下R 读的权限
14：@dev用户组对master开头的分支拥有读/写权限
15：dev1这个用户对dev开头的分支拥有读/写/强制更新的权限
16：dev2这个用户对于wip分支（严格匹配）具有读/写权限

####远程创建/删除仓储

##### 创建
关于创建仓储，方法有三种：
* 登录远程服务器创建
	ssh登录服务器，切换至git用户，进入相关目录，创建某仓储
	
	```bash
	$ mkdir somegit.git
	$ cd somegit.git
	$ git init --bare
	```

	创建完毕

* 修改gitolite.conf创建仓储
	打开gitolite-admin/conf/gitolite.conf，添加：

	```
	repo testing2
	    RW+    =  @all
	```

	保存修改，提交。
	gitolite会自动检测配置文件，发现目前没有的仓储会自动才创建。

* 通配符创建
	对于通配符版本库，即repo Projects/.+$类型的，在有创建权限的用户shell中，本地执行：
	
	```bash
	$ mkdir somegit
	$ cd somegit
	$ git init
	$ git commit --allow-empty
	$ git remote add origin git@server:Projects/somegit.git
	$ git push origin master
	```
	
	gitolite会直接创建新的仓储。

* 复制增加
从别的地方把git版本库复制过来，再配置gitolite.conf。我一般都是在gitolite.conf配置通配符版本库，然后把git版本库复制过来。比如配置了repo Projects/.+$，然后再把别的地方的git版本库复制到Projects文件夹里。
注意一些复制的文件的拥有者和群组。

	```bash
	$ chown -R git:git /home/git/repositories/Projects/android
	```

##### 删除
* 在conf/gitolite.conf中删除相关仓储配置信息（gitolite不会自动删除服务器上的文件，这点与add不同）；
* 登录服务器删除需要删除的仓储。




更多gitolit信息，请参考[Gitolite构建Git服务器](http://www.ossxp.com/doc/git/gitolite.html)

