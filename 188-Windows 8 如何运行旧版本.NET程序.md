把Win8安装盘塞到光驱里（加载到虚拟光驱也行），然后再cmd下执行下面的命令：（注意将 **X:** 换成你的光驱盘符）

	dism /online /enable-feature /featurename:NETFX3 /source:X:Sourcessxs /LimitAccess

然后耐心等待，等不及了就强行结束，重复上面的命令，等100%后就大功告成了，再打开你的.NET 3.5程序试试？