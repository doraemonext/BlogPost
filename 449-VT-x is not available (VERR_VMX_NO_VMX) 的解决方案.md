在黑苹果下想装个虚拟机，下好 VirtualBox，结果在启动系统的时候提示

> VT-x is not available (VERR_VMX_NO_VMX)

解决方案如下：

打开 `~/Virtualbox VMs/yourmachine/yourmachine.vbox` 文件，修改其中的

	<HardwareVirtEx enabled="true" exclusive="true"/>

为

	<HardwareVirtEx enabled="false"/>

修改完后，为了防止 VirtualBox 再次修改这个文件，右键这个文件，在显示简介里直接把这个文件锁上即可。