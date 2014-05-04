## 部署说明

目前我和另一个小伙伴正在进行一个微信公众号的开发，环境是 Python + Django， 而开发过程中需要一个测试服务器，需要该服务器可以支持多 Django 站点及多域名绑定，另外为了操作数据库方便，安装了 phpMyAdmin 辅助操作。

所以最后服务器可以支持多 Django 及 PHP 站点，且每个 Django 站点使用 virtualenv 来隔离 packages。

## 部署环境

直接用了一台按量付费的阿里云服务器做的实验，系统是 Ubuntu 12.04 64位，以下操作均在此环境上进行

## 安装基础环境

### 更新源

	apt-get update

### 编辑器安装

个人喜好，此步随意，下面的编辑文件的代码均使用 Emacs。

	apt-get install emacs

### 安装 Nginx 和 uWSGI

	apt-get install nginx uwsgi uwsgi-plugin-python

### 安装 Python 相关环境

	apt-get install python-dev python-virtualenv python-pip

### 安装 MySQL

	apt-get install mysql-server mysql-client libmysqld-dev

* 安装 MySQL 的过程中需要输入密码

* 安装完 MySQL 后需要执行 `mysql_secure_installation` 命令来提高 MySQL 安全性，照提示进行

### 安装 PHP

	apt-get install php5-cli php5-cgi php5-fpm php5-mcrypt php5-mysql

## 配置 Django 及多站点支持

这里假设我的网站目录为 `/www` ，每个用户有自己一个独立的文件夹（这里使用两个用户名作为示例，分别为 `test1` 和 `test2` ）

前面的操作步骤为简单起见，只新建一个站点，文件夹名称为 `test1`

### 建立目录

	mkdir /www
	mkdir /www/test1
	mkdir /www/test1/media
	mkdir /www/test1/static

这里建立的 `media` 和 `static` 目录分别用来存放该 Django 项目的 media 和 static 文件。

### 用 virtualenv 创建虚拟环境并新建测试项目

	cd /www/test1
	virtualenv env
	source env/bin/activate
	pip install Django
	django-admin.py startproject django_test

这里在 `/www/test1/env` 目录下新建了一个虚拟环境并激活，安装了 Django 并新建了一个项目，名称为 `django_test` 。

### 配置 Nginx

在 Nginx 的 `sites-available` 下创建一个新的配置文件

	emacs /etc/nginx/sites-available/test1

文件内容如下：

	server {
		listen  80;
		server_name test1.yourdomain.com;
		access_log /var/log/nginx/test1.access.log;
		error_log /var/log/nginx/test1.error.log;

		location / {
			uwsgi_pass  unix:///tmp/test1.sock;
			include     uwsgi_params;
		}

		location /media/  {
			alias /www/test1/media/;
		}

		location  /static/ {
			alias /www/test1/static/;
		}
	}

接下来将 `sites-available` 文件夹中刚才添加的文件 ln 到 `sites-enabled` 文件夹中

	ln -s /etc/nginx/sites-available/test1 /etc/nginx/sites-enabled/

### 配置 uWSGI

接下来是配置 uWSGI，同样在 uWSGI 的目录下创建一个新的配置文件

	emacs /etc/uwsgi/apps-available/test1.ini

文件内容如下

	[uwsgi]
	vhost = true
	plugins = python
	socket = /tmp/test1.sock
	master = true
	enable-threads = true
	processes = 4
	wsgi-file = /www/test1/django_test/django_test/wsgi.py
	virtualenv = /www/test1/env
	chdir = /www/test1/django_test
	touch-reload = /www/test1/django_test/reload

> **注意**：文件内容中的 `django_test` 为刚才创建的 Django 的项目名称

然后将 `apps-available` 中的配置文件 ln 到 `apps-enabled` 文件夹中

	ln -s /etc/uwsgi/apps-available/test1.ini /etc/uwsgi/apps-enabled/

### 重启 Nginx 和 uWSGI

	service uwsgi restart
	service nginx restart

接下来访问你在上面绑定的 `server_name` 域名就可以看见 It worked! 的界面了。

> **注意**：每次更新代码后需要执行 `service uwsgi restart` 修改方能生效，或者通过上面配置的 `touch-reload` 文件来检测改动更新代码。

### 多站点配置

如果需要配置多 Django 站点，只需要重复上面的操作即可。

简略演示一下建立第二个站点 `test2` 的步骤

	mkdir /www/test2
	mkdir /www/test2/media
	mkdir /www/test2/static

	cd /www/test2
	virtualenv env
	source env/bin/activate
	pip install Django
	django-admin.py startproject django_test

	emacs /etc/nginx/sites-available/test2
	仿照上面的内容填充配置文件，绑定的域名需要不同
	ln -s /etc/nginx/sites-available/test2 /etc/nginx/sites-enabled/

	emacs /etc/uwsgi/apps-available/test2.ini
	仿照上面的内容填充配置文件
	ln -s /etc/uwsgi/apps-available/test2.ini /etc/uwsgi/apps-enabled/

	service uwsgi restart
	service nginx restart

## 安装 phpMyAdmin

首先为 phpMyAdmin 建立一个独立的目录

	mkdir /www/phpmyadmin

然后将 phpMyAdmin 的源码上传到该文件夹下，接下来就是配置 Nginx 使得对应的请求可以转发至 PHP 上

### 配置 Nginx

在 Nginx 的 `sites-available` 目录下新建一个配置文件

	emacs /etc/nginx/sites-available/phpmyadmin

在其中写入如下配置

	server {
		listen 80;
		server_name phpmyadmin.yourdomain.com;

		root /www/phpmyadmin;
		index index.html index.htm index.php;

		location / {
			try_files $uri $uri/ /index.html;
		}

		location ~ \.php$ {
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			include fastcgi_params;
		}
	}

然后将其链接至 `sites-enabled` 目录下

	ln -s /etc/nginx/sites-available/phpmyadmin /etc/nginx/sites-enabled/

重启服务即可

	service php5-fpm restart
	service nginx restart

现在访问刚才绑定的域名应该可以正常访问 phpMyAdmin 了。

## 尾声

折腾这个东西折腾了不短的时间，简短的记录了下来，希望对大家有所帮助，有错误欢迎扔砖 :grin: