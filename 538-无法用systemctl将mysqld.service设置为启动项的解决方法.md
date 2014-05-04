当我尝试 `systemctl enable mysqld.service` 的时候，总是会提示： `Failed to issue method call: No such file or directory`

在stackoverflow上搜索到一个解决方案：

-----

mysqld.service is a "virtual" unit – it doesn't exist on the filesystem, it's just part of systemd's compatibility layer. You can start it and systemd will run the legacy /etc/rc.d/mysqld initscript, but you cannot systemctl enable it because you need a real .service file which could be symlinked into the proper place. You can write such a unit yourself and put it in /etc/systemd/system/mysqld.service:

	[Unit]
	Description=MySQL Server
	After=network.target

	[Service]
	ExecStart=/usr/bin/mysqld --defaults-file=/etc/mysql/my.cnf --datadir=/var/lib/mysql --socket=/var/run/mysqld/mysqld.sock User=mysql
	Group=mysql
	WorkingDirectory=/usr

	[Install]
	WantedBy=multi-user.target

Run `systemctl daemon-reload` after creating/modifying.