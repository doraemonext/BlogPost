**转载自：<http://www.cnblogs.com/gamerh2o/p/3174229.html>**

首先挂载要修复的 Fedora 系统的 `/boot` 分区，如果没有给 `/boot` 单独分区，那就挂载主分区，这里假设挂载到了 `/mnt` 目录，然后执行命令

	grub2-install --no-floppy --boot-directory=/mnt/boot /dev/sdX

上述命令是没有给 `/boot` 单独分区时的情况，如果挂载的是 `/boot` 分区，前面的 `/mnt/boot` 就要改成 `/mnt` ， `/dev/sdX` 代表系统所在的磁盘（不是分区）。更详细的说明可以用 `grub2-install --help` 命令查看。

假设上述命令执行成功，接下来要更新 grub2 的配置文件，grub2 的配置文件是用命令生成的，无需手动配置。Fedora 上没有 Ubuntu 的命令 `update-grub` ，要用命令

	grub2-mkconfig -o /mnt/boot/grub2/grub.cfg

上述是 `/boot` 没有单独分区时的命令， `/mnt` 挂载的是 `/boot` 分区的话就是 `grub2-mkconfig -o /mnt/grub2/grub.cfg` ，同理生成 EFI 启动方式需要的配置文件：

	grub2-mkconfig -o /mnt/boot/efi/EFI/fedora/grub.cfg

一切正常的话就能正常启动系统了。

如果不能正常启动，进入到了 grub 提示符中，先用 `set` 命令查看环境变量看是否有问题，不正确的环境变量可以用 `set` 命令修改，然后用 `linux (hd0,3)/boot/vmlinuz-* root=/dev/sdxX` 指定要启动的内核， `initrd (hd0,3)/boot/initramfs-*` 指定内核对应的 `initramfs` ，然后用 `boot` 命令启动设定的内核。上述命令需要根据自己的情况修改。