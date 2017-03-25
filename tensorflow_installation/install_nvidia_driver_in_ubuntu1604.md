Ubuntu 16.04 安装英伟达（Nvidia）显卡驱动
-------------------------------------


配有英伟达显卡的主机，装完 Ubuntu 16.04 后出现闪屏现象，是由于没有安装显卡驱动。


**显卡型号**  
NVIDIA Corporation GM204 [GeForce GTX 970]


由于没有显卡驱动，屏幕闪屏，以下安装过程在终端模式下进行(按 Ctrl+Alt+F1 切换到终端界面)


#### **安装过程**
##### 1. 驱动安装文件下载
* 找一台可用的机器，从 Nvidia [官网](http://www.nvidia.com/Download/index.aspx?lang=cn)下载显卡对应的驱动安装文件
		
		NVIDIA-Linux-x86_64-361.45.11.run

* 将下载到的 NVIDIA-Linux-x86_64-361.45.11.run 文件拷贝到待安装驱动的主机


##### 2. 准备工作

在待安装驱动的主机上打开一个终端(Ctrl+Alt+T)，或者直接切换到终端界面(Ctrl+Alt+F1),进行如下操作


* 卸载可能存在的旧版本 nvidia 驱动（对没有安装过 nvidia 驱动的主机，这步可以省略，但推荐执行，无害）

		$sudo apt-get remove --purge nvidia*


* 安装驱动可能需要的依赖(可选)

 		$sudo apt-get update

		$sudo apt-get install dkms build-essential linux-headers-generic

* 把 nouveau 驱动加入黑名单

		$sudo nano /etc/modprobe.d/blacklist-nouveau.conf

		在文件 blacklist-nouveau.conf 中加入如下内容：
		blacklist nouveau
		blacklist lbm-nouveau
		options nouveau modeset=0
		alias nouveau off
		alias lbm-nouveau off

* 禁用 nouveau 内核模块

		$echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf

		$sudo update-initramfs -u

* 重启


##### 3. 运行驱动安装文件


* 重启后再次进入字符终端界面，并关闭图形界面

		$sudo service lightdm stop

* 安装驱动

		$sudo chmod u+x NVIDIA-Linux-x86_64-361.45.11.run

		$sudo ./NVIDIA-Linux-x86_64-361.45.11.run

* 重启


##### 4. 安装完毕


##### 参考


[1]http://askubuntu.com/questions/760374/ubuntu-16-04-nvidia-driver-blank-screen


[2]http://www.allaboutlinux.eu/remove-nouveau-and-install-nvidia-driver-in-ubuntu-15-04/

