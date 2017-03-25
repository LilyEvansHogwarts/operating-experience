Ubuntu 16.04 安装 NVIDIA CUDA Toolkit 7.5
-------------------------------------


NVIDIA CUDA Toolkit 7.5 目前官方只适配了 Ubuntu 15.04 和 Ubuntu 14.04 两个版本的系统。如果你是这两个系统，可以直接去官方下载安装文件安装。


由于其没有适配 Ubuntu 16.04 ，这里我们通过下载官方为 CUDA 7.5 为 Uubuntu 15.04 适配的 runfile(local) 安装。


**显卡型号**  
NVIDIA Corporation GM204 [GeForce GTX 970]


#### **安装过程**
##### 1. 驱动安装文件下载
* 从 Nvidia [官网](https://developer.nvidia.com/cuda-downloads)下载 CUDA 7.5 为为 Uubuntu 15.04 适配的 runfile(local)
		
		cuda_7.5.18_linux.run


##### 2. 准备工作

打开一个终端(Ctrl+Alt+T)，或者直接切换到终端界面(Ctrl+Alt+F1),进行如下操作


* 安装可能需要的依赖

 		$sudo apt-get update
	
		$sudo apt-get install ca-certificates-java default-jre default-jre-headless fonts-dejavu-extra freeglut3 freeglut3-dev java-common libatk-wrapper-java libatk-wrapper-java-jni  libdrm-dev libgl1-mesa-dev libglu1-mesa-dev libgnomevfs2-0 libgnomevfs2-common libice-dev libpthread-stubs0-dev libsctp1 libsm-dev libx11-dev libx11-doc libx11-xcb-dev libxau-dev libxcb-dri2-0-dev libxcb-dri3-dev libxcb-glx0-dev libxcb-present-dev libxcb-randr0-dev libxcb-render0-dev libxcb-shape0-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb1-dev libxdamage-dev libxdmcp-dev libxext-dev libxfixes-dev libxi-dev libxmu-dev libxmu-headers libxshmfence-dev libxt-dev libxxf86vm-dev lksctp-tools mesa-common-dev x11proto-core-dev x11proto-damage-dev x11proto-dri2-dev x11proto-fixes-dev x11proto-gl-dev x11proto-input-dev x11proto-kb-dev x11proto-xext-dev x11proto-xf86vidmode-dev xorg-sgml-doctools xtrans-dev libgles2-mesa-dev nvidia-modprobe build-essential


##### 3. 安装 CUDA

由于我已经安装过 NVIDIA 最新版显卡驱动（[安装 Nvidia 显卡驱动](https://gist.github.com/dangbiao1991/7825db1d17df9231f4101f034ecd5a2b)），因此安装 CUDA 过程中不再安装显卡驱动。


	$ chmod 755 cuda_7.5.18_linux.run

	$ sudo ./cuda_7.5.18_linux.run --override
	-------------------------------------------------------------
	Do you accept the previously read EULA? (accept/decline/quit): accept
	You are attempting to install on an unsupported configuration. Do you wish to continue? ((y)es/(n)o) [ default is no ]: y
	Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 352.39? ((y)es/(n)o/(q)uit): n
	Install the CUDA 7.5 Toolkit? ((y)es/(n)o/(q)uit): y
	Enter Toolkit Location [ default is /usr/local/cuda-7.5 ]:
	Do you want to install a symbolic link at /usr/local/cuda? ((y)es/(n)o/(q)uit): y
	Install the CUDA 7.5 Samples? ((y)es/(n)o/(q)uit): y
	Enter CUDA Samples Location [ default is /home/kinghorn ]: /usr/local/cuda-7.5
	Installing the CUDA Toolkit in /usr/local/cuda-7.5 ...
	Finished copying samples.

	===========
	= Summary =
	===========

	Driver:   Not Selected
	Toolkit:  Installed in /usr/local/cuda-7.5
	Samples:  Installed in /usr/local/cuda-7.5


##### 4. 配置 CUDA 环境变量

	$sudo vim /etc/profile.d/cuda.sh

	export PATH=$PATH:/usr/local/cuda/bin
	export LD_LIBRARY_PATH=$PATH:/usr/local/cuda/lib64

	$ source /etc/profile.d/cuda.sh


##### 5. 修改 CUDA header 文件使其兼容 gcc 5

该版本的 CUDA 无法在 gcc  > 4.8 时使用，因此我们需要注释掉代码中的 gcc 版本检测代码

	sudo vim /usr/local/cuda/include/host_config.h

	line: 115 comment out error
	//#error -- unsupported GNU version! gcc versions later than 4.9 are not supported!  

##### 6. 编译 CUDA Samples

* 将 Samples 代码拷贝到一个普通用户可读写的目录（这里我们拷到根目录 ～/ ）

		$ rsync -av /usr/local/cuda/samples ~/

* 如果你在安装 CUDA 的时候安装了 Driver （此版本的驱动应该是Nvidia 352），这里可以考虑直接编译并忽略下面两步（未验证过此种方法）
		
		$ cd ~/samples/

		$ make 

* 如果你在安装 CUDA 的时候没有 Driver ，而是安装了最新版的 Nvidia 361 显卡驱动,你需要修改 samples 代码中引用到的库的路径，然后再编译
		
		$ cd ~/samples/

		$ grep -r nvidia-352 -l --null . | xargs -0 sed -i 's#nvidia-352#nvidia-361#g'

		$ make

* 编译的过程中你可能会遇到找不到 nvidia-361 库的提示，该问题可以通过在 apt 中安装 第三方 nvidia-361 解决。但是经实验，安装完成后虽然可以编译 samples ，但是运行 samples 时提示没有显卡驱动，于是再重装官方显卡驱动，问题解决。，切换到终端界面(Ctrl+Alt+F1),进行如下操作

		$ sudo service lightdm stop

		$ sudo apt-get install nvidia-361

		$ sudo reboot

	切换到终端界面(Ctrl+Alt+F1)：

		$ sudo service lightdm stop

		$ sudo ./NVIDIA-Linux-x86_64-361.45.11.run


NVIDIA-Linux-x86_64-361.45.11.run 文件的下载方法见[Ubuntu 16.04 安装英伟达（Nvidia）显卡驱动](https://gist.github.com/dangbiao1991/7825db1d17df9231f4101f034ecd5a2b)



##### 7. gcc 5 的编译问题

由于 gcc 5 用到的 /usr/include/string.h 相比 gcc 4 中的 string.h 有所改动。在编译samples的过程中你可能会遇到类似这种错误

	/usr/include/string.h: In function ‘void* __mempcpy_inline(void*, const void*, size_t)’: /usr/include/string.h:652:42: error: ‘memcpy’ was not declared in this scope return (char *) memcpy (__dest, __src, __n) + __n; ^ /usr/include/string.h: In function ‘void* __mempcpy_inline(void*, const void*, size_t)’: /usr/include/string.h:652:42: error: ‘memcpy’ was not declared in this scope return (char *) memcpy (__dest, __src, __n) + __n; 

解决此问题的方法是在对应的 sample 目录下的 Makefile 中增加编译选项 -D_FORCE_INLINES 既可解决


例如将 Makefile 中的

	NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)

替换为

	NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS).


##### 8. 运行 samples 

	$ cd ~/samples/bin/x86_64/linux/release

	$ ./nbody -benchmark -numbodies=256000

	Windowed mode
	Simulation data stored in video memory
	Single precision floating point simulation
	1 Devices used for simulation
	GPU Device 0: "GeForce GTX 970" with compute capability 5.2

	Compute 5.2 CUDA device: [GeForce GTX 970]
	number of bodies = 256000
	256000 bodies, total time for 10 iterations: 7207.025 ms
	= 90.934 billion interactions per second
	= 1818.670 single-precision GFLOP/s at 20 flops per interaction


##### 参考


[1]https://www.pugetsystems.com/labs/articles/NVIDIA-CUDA-with-Ubuntu-16-04-beta-on-a-laptop-if-you-just-cannot-wait-775/


[2]https://github.com/BVLC/caffe/issues/4046

