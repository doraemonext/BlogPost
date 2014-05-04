第一种是取回一个整数的返回值

	#!/bin/sh

	function() {
		echo "function"
		return 1;
	}

	function
	x=$?
	echo $x

	exit 0

第二种是取回一个字符串的返回值

	#!/bin/sh

	function() {
		echo "function"
	}

	x=$(function)
	echo $x

	exit 0
