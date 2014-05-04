## Apache

	sudo apt-get install apache2

## PHP

	sudo apt-get install php5

## MySQL

	sudo apt-get install mysql-server

安装过程中需要输入密码

## MySQL for Apache PHP

	sudo apt-get install libapache2-mod-auth-mysql php5-mysql

## phpMyAdmin

	sudo apt-get install phpmyadmin

选择apache2，并输入刚才安装MySQL时输入的密码

安装后，网站默认目录是/var/www，先加上权限

	sudo chmod -R 777 /var/www

然后将phpMyAdmin复制到网站目录中即可完成安装

	cp -r /usr/share/phpmyadmin /var/www