Emacs For Max OS X的安装并没有将终端中的Emacs修改掉，还是那个22版本的，下面的方法可以将其替换成我们所安装的最新版本。 在 `/bin` 目录下新建一个名为 `emacs` 的脚本

	sudo emacs /bin/emacs

在脚本中添加以下内容：

	#!/bin/sh
	exec /Applications/Emacs.app/Contents/MacOS/Emacs "$@"

然后添加权限：

	sudo chmod +x /bin/emacs

下面就是将默认的 `emacs` 删掉了

	sudo mv /usr/bin/emacs /usr/bin/emacs.backup

重新启动终端后emacs便已经更新成功了