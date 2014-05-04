最简单的参数赋值和扩展：

	foo=fred
	echo $foo

当想编写一个脚本程序来处理连续的`1_tmp`, `2_tmp`, `3_tmp`这种文件的时候，可以这样写：

	#!/bin/sh

	for i in 1 2 3
	do
		process ${i}_tmp
	done

为i加上 `{}` 即可保证每次会将i进行替换，否则只会传递一个无意义的空值i_tmp。

一些其他的精巧的参数扩展的方法：

| 参数扩展             |	说明
|---------------------|----------------------------------------------------------------------
| `${param:-default}` |	如果param为空，就返回default的值（param不变）
| `${param:=default}` |	如果param为空，就将param设置为default的值，然后返回default的值
| `${param:?default}` |	在param不存在或它设置为空的情况下，输出param:default并异常终止脚本程序
| `${param:+default}` |	在param存在且不为空的情况下返回default
| `${#param}`         |	给出param的长度
| `${param%word}`     |	从param的尾部开始删除与word匹配的最小部分，然后返回剩余部分
| `${param%%word}`    |	从param的尾部开始删除与word匹配的最长部分，然后返回剩余部分
| `${param#word}`     |	从param的头部开始删除与word匹配的最小部分，然后返回剩余部分
| `${param##word}`    |	从param的头部开始删除与word匹配的最长部分，然后返回剩余部分

示例程序：

	#!/bin/sh

	unset foo
	echo ${foo:-bar}
	echo $foo
	echo ${foo:=bar}
	echo $foo

	foo=fud
	echo ${foo:-bar}

	foo=/usr/bin/X11/startx
	echo ${foo#*/}
	echo ${foo##*/}

	bar=/usr/local/etc/local/networks
	echo ${bar%local*}
	echo ${bar%%local*}

	length=10
	echo ${#length}

	exit 0

输出结果如下：

	bar

	bar
	bar
	fud
	usr/bin/X11/startx
	startx
	/usr/local/etc/
	/usr/
	2
