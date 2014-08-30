---
layout: post
title: 基于CentOS的CloudStack快速安装指南
category: learn
tags: [cloudstack]

---

### 概述
#### 我们真正要构建什么？

构建基础架构即服务(IaaS)云是一个很复杂的工作，它定义了过多的选项，甚至新接触云平台的管理员都很容易混乱。此篇操作手册的目的就是为减少安装CloudStack时所发生的问题并提供最直接的帮助。

<!--break-->

##### 整体过程概述

该操作手册将重点介绍如何搭建Cloudstack云平台: 使用CentOS 6.4作为KVM和NFS存储主机，部署于扁平二层网络并使用三层网络隔离(安全组), 所有资源集中于一台物理主机

KVM或Kernel-based Virtual Machine是一种基于LInux内核的虚拟化技术。KVM支持本地虚拟化，主机的CPU处理器需支持硬件虚拟化扩展。

安全组起到类似分布式防火墙的作用，它可以对一组虚拟机进行访问控制。

#### 先决条件
完成此操作手册你需要如下条件：

1. 至少一台支持硬件虚拟化的主机。
2. The CentOS 6.5 x86_64 minimal install CD
3. 一个C类(/24)网络，网关为 xxx.xxx.xxx.1，网络中不能存在DHCP服务器，所有运行Cloudstack的主机需使用静态IP地址。

#### 环境
首先，需要先准备要Cloudstack的安装环境，以下将详细描述准备步骤。

##### 操作系统
Using the CentOS 6.5 x86_64 minimal install ISO, you’ll need to install CentOS on your hardware. The defaults will generally be acceptable for this installation.

当安装完成后，需要以root身份通过SSH连接新安装的主机，注意不要以root账户登录生产环境，请在完成安装和配置后关闭远程登录。

##### 网络配置
默认情况下新安装的机器并未启用网络，您需要根据实际环境进行配置。由于网络中不存在DHCP服务器，您需要手工配置网络接口。为了实现快速简化安装的目标，这里假定主机上只有eth0一个网络接口。

使用root用户登录本地控制台。检查文件 /etc/sysconfig/network-scripts/ifcfg-eth0，默认情况，其内容如下所示：

	DEVICE="eth0"
	HWADDR="52:54:00:B9:A6:C0"
	NM_CONTROLLED="yes"
	ONBOOT="no"

但是根据以上配置您无法连接到网络，对于Cloudstack也同样不适合；您需修改配置文件，指定IP地址，网络掩码等信息，如下例所示：


注意，不要在你的配置中使用示例中的Hardware地址(也叫MAC地址)。该地址为网络接口所特有的，请保留你配置文件中已经提供的HWADDR字段。

	DEVICE=eth0
	HWADDR=52:54:00:B9:A6:C0
	NM_CONTROLLED=no
	ONBOOT=yes
	BOOTPROTO=none
	IPADDR=172.16.10.2
	NETMASK=255.255.255.0
	GATEWAY=172.16.10.1
	DNS1=8.8.8.8
	DNS2=8.8.4.4

配置文件准备完毕后，需要运行命令启动网络。

	# chkconfig network on
	# service network start

##### 主机名
Cloudstack要求正确设置主机名，如果安装时您接受了默认选项，主机名为localhost.localdomain，输入如下命令可以进行验证

	# hostname --fqdn
在此处将返回：

	localhost
为了纠正这个问题，需设置主机名，通过编辑/etc/hosts 文件，将其更改为类似如下内容：

	127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
	172.16.10.2 srvr1.cloud.priv
更改配置文件后，重启网络服务：

	# service network restart
通过hostname –fqdn命令重新检查主机名，并确认返回了正确的FQDN

##### SELinux
当前的CloudStack需要将SELinux设置为permissive才能正常工作，您需要改变当前配置，同时将该配置持久化，使其在主机重启后仍然生效。

在系统运行状态下将SELinux配置为permissive需执行如下命令：

	# setenforce 0
为确保其持久生效需更改配置文件/etc/selinux/config，设置为permissive，如下例所示：

	# This file controls the state of SELinux on the system.
	# SELINUX= can take one of these three values:
	# enforcing - SELinux security policy is enforced.
	# permissive - SELinux prints warnings instead of enforcing.
	# disabled - No SELinux policy is loaded.
	SELINUX=permissive
	# SELINUXTYPE= can take one of these two values:
	# targeted - Targeted processes are protected,
	# mls - Multi Level Security protection.
	SELINUXTYPE=targeted


##### NTP
为了同步云平台中主机的时间，需要配置NTP，但NTP默认没有安装。因此需要先安装NTP，然后进行配置。通过以下命令进行安装：

	# yum -y install ntp
实际上默认配置项即可满足的需求，仅需启用NTP并设置为开机启动，如下所示：

	# chkconfig ntpd on
	# service ntpd start

##### 配置ClouStack软件库

配置主机使用CloudStack软件库。

添加CloudStack软件仓库，创建/etc/yum.repos.d/cloudstack.repo文件，并添加如下信息。

	[cloudstack]
	name=cloudstack
	baseurl=http://cloudstack.apt-get.eu/rhel/4.4/
	enabled=1
	gpgcheck=0

##### NFS
本文档将配置的环境使用NFS做为主存储和辅助存储，需配置两个NFS共享目录，在此之前需先安装nfs-utils：

	# yum install nfs-utils
接下来需配置NFS提供两个不同的挂载点。通过编辑/etc/exports文件即可简单实现。确保这个文件中包含下面内容：

	/secondary *(rw,async,no_root_squash,no_subtree_check)
	/primary *(rw,async,no_root_squash,no_subtree_check)
注意配置文件中指定了系统中两个并不存在的目录，下面需要创建这些目录并设置合适的权限，对应的命令如下所示：

	# mkdir /primary
	# mkdir /secondary
CentOS 6.x 版本默认使用NFSv4，NFSv4要求所有客户端的域设置匹配，这里以设置cloud.priv为例，请确保文件/etc/idmapd.conf中的域设置没有被注释掉，并设置为以下内容：

在/etc/sysconfig/nfs文件中取消如下选项的注释：

	LOCKD_TCPPORT=32803
	LOCKD_UDPPORT=32769
	MOUNTD_PORT=892
	RQUOTAD_PORT=875
	STATD_PORT=662
	STATD_OUTGOING_PORT=2020
接下来还需配置防火墙策略，允许NFS客户端访问。编辑文件/etc/sysconfig/iptables

	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 111 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 111 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 2049 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 32803 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 32769 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 892 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 892 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 875 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 875 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 662 -j ACCEPT
	-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 662 -j ACCEPT
通过以下命令重新启动iptables服务：

	# service iptables restart
最后需要配置NFS服务为开机自启动，执行如下命令：

	# service rpcbind start
	# service nfs start
	# chkconfig rpcbind on
	# chkconfig nfs on

##### 管理服务器安装

接下来进行CloudStack管理节点和相关工具的安装。

数据库安装和配置
首先安装MySQL，并对它进行配置，以确保CloudStack运行正常。

运行如下命令安装：

	# yum -y install mysql-server
MySQL安装完成后，需更改其配置文件/etc/my.cnf。在[mysqld]下添加下列参数：

	innodb_rollback_on_timeout=1
	innodb_lock_wait_timeout=600
	max_connections=350
	log-bin=mysql-bin
	binlog-format = 'ROW'
正确配置MySQL后，启动它并配置为开机自启动：

	# service mysqld start
	# chkconfig mysqld on
安装
现在将要开始安装管理服务器。执行以下命令：

	# yum -y install cloudstack-management
在程序执行完毕后，需初始化数据库，通过如下命令和选项完成：

	# cloudstack-setup-databases cloud:password@localhost --deploy-as=root:mysqlpasswd
当该过程结束后，您应该可以看到类似信息：”CloudStack has successfully initialized the database.”。

其中mysqlpasswd是mysql的root密码。
可以用下面的命令来设置

	# mysql_secure_installation

数据库创建后，最后一步是配置管理服务器，执行如下命令：

	# cloudstack-setup-management

##### 上传系统模板

CloudStack通过一系列系统虚拟机提供功能，如访问虚拟机控制台，如提供各类网络服务，以及管理辅助存储的中的各类资源。该步骤会获取系统虚拟机模板，用于云平台引导后系统虚拟机的部署。

然后需要下载系统虚拟机模板，并把这些模板部署于刚才创建的辅助存储中；管理服务器包含一个脚本可以正确的操作这些系统虚拟机模板：

	/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
	-m /secondary \
	-u http://cloudstack.apt-get.eu/systemvm/4.4/systemvm64template-4.4.0-6-kvm.qcow2.bz2 \
	-h kvm -F
以上是管理服务器的安装和配置过程；在配置CloudStack之前，仍需配置hypervisor。

#### KVM配置和安装
本文档使用KVM作为hypervisor，下文将回顾最如何配置hypervisor主机，其中大部分配置工作已在配置管理节点时完成；接下来描述如何安装agent。您可以应用相同的步骤添加额外的KVM节点到CloudStack环境中。

##### 先决条件
本文档描述的环境使用管理服务器同时作为计算节点，这意味着很多先决步骤已经在搭建管理服务器时完成；但为了清晰起见，仍然列出相关步骤：

1. 网络配置
2. 主机名
3. SELinux
4. NTP
5. 配置ClouStack软件库

你不需要在管理节点上执行这些操作，当然，如果您需要添加额外的主机以上步骤仍然需要执行。

##### 安装
安装KVM代理仅仅需要一条简单的命令，但之后我们需要进行一些配置。

	# yum -y install cloudstack-agent
配置KVM
KVM中我们有两部分需要进行配置, libvirt和QEMU.

##### 配置QEMU
KVM的配置项相对简单，仅需配置一项。编辑QEMU VNC配置文件/etc/libvirt/qemu.conf并取消如下行的注释。

	vnc_listen=0.0.0.0
##### 配置Libvirt
CloudStack使用libvirt管理虚拟机。因此正确的配置libvirt至关重要。Libvirt属于cloudstack-agent的依赖组件，应提前安装好。

为了实现动态迁移，libvirt需要监听使用非加密的TCP连接。还需要关闭libvirts尝试使用组播DNS进行广播。这些都是在 /etc/libvirt/libvirtd.conf文件中进行配置。

设置下列参数：

	listen_tls = 0
	listen_tcp = 1
	tcp_port = "16059"
	auth_tcp = "none"
	mdns_adv = 0
仅仅在libvirtd.conf中启用”listen_tcp”还不够，我们还必须修改/etc/sysconfig/libvirtd中的参数:

取消如下行的注释：

	#LIBVIRTD_ARGS="--listen"
重启libvirt服务

	# service libvirtd restart

#### KVM配置完成
最后一步是配置管理服务器，执行如下命令：

	# cloudstack-setup-agent

以上内容是针对KVM的安装和配置，下面将介绍如何使用CloudStack用户界面配置云平台。

####CloudStack用户界面配置云平台

#####配置
如上文所述，该手册所描述的环境将使用安全组提供网络隔离，这意味着您的安装环境仅需要一个扁平的二层网络，同样意味着较为简单的配置和快速的安装。

####访问用户界面
要访问CloudStack的WEB界面，仅需在浏览器访问 http://172.16.10.2:8080/client ，使用默认用户’admin’和密码’password’来登录。第一次登录可以看到欢迎界面，提供两个选项设置CloudStack。请选择继续执行基本配置安装。

此时您会看到提示，要求为admin用户更改密码，请更新密码后继续。

##### 配置区域
区域是Cloudstack中最大的组织单位，下面将要讲述如何创建，此时屏幕中显示的是区域添加页面，这里需要您提供5部分信。

	名称 - 提供描述性的名称，这里以”Zone1”为例
	公共DNS1 - 我们输入8.8.8.8。
	公共DNS 2 - 我们输入8.8.4.4。
	内部DNS 1 - 我们同样输入8.8.8.8。
	内部DNS 2 - 我们同样输入8.8.4.4。

CloudStack分为内部和公共DNS。内部DNS只负责解析内部主机名，比如NFS服务器的DNS名称。公共DNS为虚拟机提供公共IP地址解析。你可以指定同一个DNS服务器，但如果这样做，你必须确保内部和公共IP地址都能路由到该DNS服务器。在我们的案例中对内部资源不使用DNS名称，因此这里将其设置为与外部DNS一致用以简化安装，从而不必为此再安装一台DNS服务器。

#####配置提供点

到这里我们已经添加了一个区域，下一步会显示添加提供点所需信息：

	提供点名称 - 使用Pod1
	网关 - 使用172.16.10.1作为网关
	子网掩码 - 使用255.255.255.0
	系统预留 起始/结束 IP - 使用172.16.10.10-172.16.10.20
	来宾网络网关 - 使用172.16.10.1
	来宾网络掩码 - 使用255.255.255.0
	来宾网络 起始/结束IP - 使用172.16.10.30-172.16.10.200

#####群集

添加区域后，仅需再为配置群集提供如下信息。

	群集名称 - 使用Cluster1
	Hypervisor - 选择KVM
此时向导会提示您为集群添加第一台主机，需提供如下信息：

	主机名 - 由于没有设置DNS服务器，所以这里输入主机IP地址 172.16.10.2。
	用户名 - 我们使用root
	密码 - 输入root用户的密码

##### 主存储
在配置群集时，按提示输入主存储的信息。选择NFS作为存储类型，并在如下区域中输入相应的值：

	名称 - 使用 ‘Primary1’
	服务器 - 使用的IP地址为 172.16.10.2
	路径 - 使用之前定义的 /primary

####辅助存储

如果添加的是一个新的区域，您需提供辅助存储相关信息 - 如下所示：

	NFS服务器 - 使用的IP地址为 172.16.10.2
	路径 - 输入 /secondary

现在，点击启动开始配置云-完成安装所需的时间取决于你的网络速度。

到这里，你的Apache CloudStack云就已经安装完成了。



本文章转自[基于CentOS的快速安装指南](http://cloudstack-installation-zh-cn.readthedocs.org/zh_CN/latest/qig.html)