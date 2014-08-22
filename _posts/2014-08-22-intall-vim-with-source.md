---
layout: post
title: 源代码编译安装vim 7.4
category: learn
tags: [vim]

---

1. 卸载

	```bash
	yum remove vim vim-enhanced vim-common vim-minimal
	``` 

2. 下载、解压

	```bash
	wget ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2  
	wget ftp://ftp.vim.org/pub/vim/extra/vim-7.2-extra.tar.gz 
	wget ftp://ftp.vim.org/pub/vim/extra/vim-7.2-lang.tar.gz  
	
	tar jxvf vim-7.4.tar.bz2  
	tar zxvf vim-7.2-extra.tar.gz  
	tar zxvf vim-7.2-lang.tar.gz  
	
	mv vim72 vim74 
	``` 

	<!--break-->

3. 安装编译环境

	```bash
	yum install ncurses-devel  
	yum groupinstall "Development Tools"
	```

4. 编译
进入vim74/src

	```bash
	./configure --prefix=/usr --with-features=huge  --disable-selinux --enable-pythoninterp --enable-cscope --enable-multibyte
	make 
	make install
	ln -s /usr/local/bin/vim /usr/local/bin/vi
	```

