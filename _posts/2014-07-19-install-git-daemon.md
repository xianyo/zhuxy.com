---
layout: post
title: Git-daemon安装与配置
category: learn
tags: [git]

---

对于提供公共的，非授权的只读访问，我们可以抛弃 HTTP 协议，改用 Git 自己的协议，这主要是出于性能和速度的考虑。Git 协议远比 HTTP 协议高效，因而访问速度也快，所以它能节省很多用户的时间。

重申一下，这一点只适用于非授权的只读访问。如果建在防火墙之外的服务器上，那么它所提供的服务应该只是那些公开的只读项目。如果是在防火墙之内的服务器上，可用于支撑大量参与人员或自动系统（用于持续集成或编译的主机）只读访问的项目，这样可以省去逐一配置 SSH 公钥的麻烦。

<!--break-->

### 安装

```bash
sudo apt-get install git-daemon git-daemon-sysvinit
```

### 配置

配置文件在`/etc/default/git-daemon`  
打开编辑

```bash
sudo vi /etc/default/git-daemon
```
例子：

```
# Defaults for git-daemon initscript
# sourced by /etc/init.d/git-daemon
# installed at /etc/default/git-daemon by the maintainer scripts

#
# This is a POSIX shell fragment
#

GIT_DAEMON_ENABLE=true
GIT_DAEMON_USER=git
GIT_DAEMON_BASE_PATH=/home/git/mirror
GIT_DAEMON_DIRECTORY="/var/lib/git /home/git/mirror"

# Additional options that are passed to the Daemon.
# GIT_DAEMON_OPTIONS="--export-all --enable=upload-pack \ 
--enable=upload-archive --enable=receive-pack --informative-errors"
GIT_DAEMON_OPTIONS="--export-all --enable=upload-pack --enable=upload-archive --informative-errors"
                      
```


```
GIT_DAEMON_BASE_PATH=/home/git/mirror #设置版本库的目录
 
--export-all # 全部导出，否则要到repo下touch git-daemon-export-ok

--enable=receive-pack # 可以git push

```

```
git clone git://localhost/test.git 
```



以上设置好了以后，每次开机就会自己启动了．
git-daemon控制

```
sudo service git-daemon start|restart|sto
sudo /etc/init.d/git-daemon start|restart|stop
```
