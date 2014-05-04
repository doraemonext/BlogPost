trap用于指定在接收到信号后将要采取的行动

使用方法：

	trap command signal

如果要重置某个信号的处理方式到默认值，只需要将command设置为 `-` ，忽略某个信号，只需要将command设置为空字符串（注意是`""`或`''`，不能不填）即可。

不带参数的trap将列出当前设置的信号及其行动的清单。

**另外，在trap语句中，单引号和双引号是不同的，当shell程序第一次碰到trap语句时，将把command中的命令扫描一遍。此时若command是用单引号括起来的话，那么shell不会对command中的变量和命令进行替换，否则command中的变量和命令将用当时具体的值来替换。**

下表是比较重要的信号说明

| 信号     | 说明
|---------|---------------------------------------
| `HUP`   |	挂起，通常因终端掉线或用户退出而引发
| `INT`   |	终端，通常因按下Ctrl+C组合键而引发
| `QUIT`  |	退出，通常因按下Ctrl+组合键而引发
| `ABRT`  |	中止，通常因某些严重的执行错误而引发
| `ALRM`  |	报警，通常用来处理超时
| `TERM`  |	终止，通常在系统关机时发送

使用示例：

	#!/bin/sh

	trap 'rm -f /tmp/my_tmp_file_$$' INT
	echo "creating file /tmp/my_tmp_file_$$"
	date > /tmp/my_tmp_file_$$

	echo "press interrupt (CTRL-C) to interrupt ...."
	while [ -f /tmp/my_tmp_file_$$ ]; do
		echo "File exists"
		sleep 1
	done
	echo "The file no longer exists"

	trap INT
	echo "creating file /tmp/my_tmp_file_$$"
	date > /tmp/my_tmp_file_$$
	echo "press interrput (control-C) to interrput ...."
	while [ -f /tmp/my_tmp_file_$$ ]; do
		echo "File exists"
		sleep 1
	done

	echo "we never get here"

	exit 0
