主要修改操作都在/etc/fstab文件中，打开这个文件，准备修改。

## 删除swap空间

现在内存一般都够大，一般不会出现内存不足的现象，我就直接将和swap有关的一行删除掉了。

## 开启TRIM和设置noatime选项

我的fstab中的默认内容如下（已删除swap）：

	/dev/mapper/fedora_doraemonext--laptop-root /                       ext4    defaults        1 1
	UUID=cd9bb8dc-ddf1-4b2d-86e6-b4ae87e674a6 /boot                   ext4    defaults        1 2
	/dev/mapper/fedora_doraemonext--laptop-home /home                   ext4    defaults        1 2

将其改成

	/dev/mapper/fedora_doraemonext--laptop-root /                       ext4    noatime,discard,defaults        1 1
	UUID=cd9bb8dc-ddf1-4b2d-86e6-b4ae87e674a6 /boot                   ext4    noatime,discard,defaults        1 2
	/dev/mapper/fedora_doraemonext--laptop-home /home                   ext4    noatime,discard,defaults        1 2

## 内存分区加速

将一些经常变化的位置，如/tmp放入内存，减少对SSD的访问。

依然将下面的语句加入到/etc/fstab文件中

	tmpfs /tmp tmpfs defaults,noatime,mode=1777 0 0
	tmpfs /var/tmp tmpfs defaults,noatime,mode=1777 0 0
	tmpfs /var/log tmpfs defaults,noatime,mode=1777 0 0

剩下的就是重新启动了……虽然效果一点没看出来，不过就算是心理上的一点安慰吧……