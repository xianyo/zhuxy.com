---
layout: post
title: Cgit安装与配置
category: learn
tags: [git]

---

cgit 是一种git 代码浏览工具，类似gitweb ，但是更快速，openembedded ,freedeskop 等项目都在采用cgit

### 安装

```bash
sudo apt-get install libssl-dev apache2

sudo a2enmod rewrite
sudo a2enmod cgi

git clone git://git.zx2c4.com/cgit 
cd cgit/
git submodule init
git submodule update
make 
sudo make install
```

<!--break-->

### 配置

#### 配置apache2

/etc/apache2/sites-available/cgit.conf

```
<VirtualHost *:80>
    ServerAdmin admin@example.com
    ServerName git.example.com

    DocumentRoot /var/www/htdocs/cgit/

    <Directory /var/www/htdocs/cgit/>
        AllowOverride None
        Options +ExecCGI
        
        ### Old apache2 version
        # Order allow,deny
        # Allow from all
        
        # New authorization commands for apache 2.4 and up
        # http://httpd.apache.org/docs/2.4/upgrading.html#access
        Require all granted
    </Directory>

    Alias /cgit.png /var/www/htdocs/cgit/cgit.png
    Alias /cgit.css /var/www/htdocs/cgit/cgit.css
    ScriptAlias / "/var/www/htdocs/cgit/cgit.cgi/"
    RewriteRule ^$ / [R]
    RewriteRule ^/(.*)$ /cgit.cgi/$1 [PT]

    ErrorLog /var/log/apache2/cgit-error.log

    # Possible values include: debug, info, notice, warn, error, crit,
    # alert, emerg.
    LogLevel warn

    CustomLog /var/log/apache2/cgit-access.log combined
</VirtualHost>
```

```
sudo a2ensite cgit
sudo service apache2 reload
```

#### 配置cgit
* cgit配置文件

	/etc/cgitrc
	```
	# Enable caching of up to 1000 output entriess
	cache-size=1000
	
	# cache time to live 
	cache-dynamic-ttl=5
	cache-repo-ttl=5
	cache-repo-ttl=5
	
	# Specify some default clone urls using macro expansion
	clone-url=http://$HTTP_HOST$SCRIPT_NAME/$CGIT_REPO_URL git://$HTTP_HOST/$CGIT_REPO_URL
	
	# Specify the css url
	css=/cgit.css
	
	# Show owner on index page
	enable-index-owner=0
	
	# Source gitweb.description, gitweb.owner from each project config
	enable-git-config=1
	
	# Allow http transport git clone
	enable-git-clone=1
	
	# Show extra links for each repository on the index page
	enable-index-links=1
	
	# Remove .git suffix from project display
	remove-suffix=1
	
	# Enable ASCII art commit history graph on the log pages
	enable-commit-graph=1
	
	# Show number of affected files per commit on the log pages
	enable-log-filecount=1
	
	# Show number of added/removed lines per commit on the log pages
	enable-log-linecount=1
	
	# Sort branches by date
	branch-sort=age
	
	# Add a cgit favicon
	favicon=/favicon.ico
	
	# Use a custom logo
	logo=/cgit.png
	
	# Enable statistics per week, month and quarter
	max-stats=quarter
	
	# Set the title and heading of the repository index page
	root-title=My Git repositories
	
	# Set a subheading for the repository index page
	root-desc=tracking the foobar development
	
	# Include some more info about example.com on the index page
	root-readme=/var/www/htdocs/cgit/about.htm
	
	# Allow download of tar.gz, tar.bz2 and zip-files
	snapshots=tar.gz tar.bz2 zip
	
	##
	## List of common mimetypes
	##
	
	mimetype.gif=image/gif
	mimetype.html=text/html
	mimetype.jpg=image/jpeg
	mimetype.jpeg=image/jpeg
	mimetype.pdf=application/pdf
	mimetype.png=image/png
	mimetype.svg=image/svg+xml
	
	# Highlight source code with python pygments-based highligher
	source-filter=/usr/local/lib/cgit/filters/syntax-highlighting.sh
	
	# Format markdown, restructuredtext, manpages, text files, and html files
	# through the right converters
	about-filter=/usr/local/lib/cgit/filters/about-formatting.sh
	
	##
	## Search for these files in the root of the default branch of repositories
	## for coming up with the about page:
	##
	readme=:README.md
	readme=:readme.md
	readme=:README.mkd
	readme=:readme.mkd
	readme=:README.rst
	readme=:readme.rst
	readme=:README.html
	readme=:readme.html
	readme=:README.htm
	readme=:readme.htm
	readme=:README.txt
	readme=:readme.txt
	readme=:README
	readme=:readme
	readme=:INSTALL.md
	readme=:install.md
	readme=:INSTALL.mkd
	readme=:install.mkd
	readme=:INSTALL.rst
	readme=:install.rst
	readme=:INSTALL.html
	readme=:install.html
	readme=:INSTALL.htm
	readme=:install.htm
	readme=:INSTALL.txt
	readme=:install.txt
	readme=:INSTALL
	readme=:install
	
	##
	## List of repositories.
	## PS: Any repositories listed when section is unset will not be
	##     displayed under a section heading
	## PPS: This list could be kept in a different file (e.g. '/etc/cgitrepos')
	##      and included like this:
	##        include=/etc/cgitrepos
	##
	#project-list=/home/git/projects.list
	#scan-path=/home/git/repositories
	
	max-repo-count=100
	
	section-from-path=1
	scan-path=/home/git/mirror/
	
	```
*  高亮 

	```
	sudo apt-get install highlight
	```
	
	/usr/local/lib/cgit/filters/syntax-highlighting.sh
	
	```
	# This is for version 2
	#exec highlight --force --inline-css -f -I -X -S "$EXTENSION" 2>/dev/null
	
	# This is for version 3
	exec highlight --force --inline-css -f -I -O xhtml -S "$EXTENSION" 2>/dev/null
	```
	根据highlight的版本注释保留选择相应的选项


* 权限配置

	```
	sudo gpasswd -a www-data git
	find /path/to/the/repository/ -type d | xargs chmod g+rx
	find /path/to/the/repository/ -type f | xargs chmod g+r
	```

* gitolite相关

	```
	chmod g+r /home/git/{projects.list,.gitolite.rc}
	chmod g+rx /home/git/{repositories/,.gitolite/}
	```
	
	~/.gitolite.rc
	```
	$UMASK=0027
	GIT_CONFIG_KEYS  =>  '.*',
```
 
