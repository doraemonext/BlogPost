注意**字符串需要用双引号括起来**，否则结果不正确（比如字符串中混进个空格这个捣蛋鬼……没有引号就会BUG了……）

可以在shell下直接实验一下

	string=
	[ -z "$string" ]
	echo $?

注意返回值为0表示测试正常，比如-z测试字符串是否是空的，当字符串为空的时候，test通过，返回0，别在if那搞混了。

	#!/bin/sh

	string=
	if [ -z "$string" ]; then
		echo "STRING is empty"
	fi

	string=abcd
	if [ -n "$string" ]; then
		echo "STRING is not empty"
	fi

	exit 0
