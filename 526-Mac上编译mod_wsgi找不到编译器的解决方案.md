今天在编译安装mod_wsgi的时候直接出现了

	env: /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr/bin/cc: No such file or directory
	apxs:Error: Command failed with rc=65536
	.
	make: *** [mod_wsgi.la] Error 1

网上搜了一下解决方案，其实这个东西是有的，只不过不是在它写的这个位置，所以，最快的解决方法就是直接用 `ln` 链接过去：

	sudo ln -s /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/ /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain

> **注意**：我的系统是Mountain Lion，如果你的系统是Mavericks，那么就是下面的这条命令：

> `sudo ln -s /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/ /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.9.xctoolchain`
