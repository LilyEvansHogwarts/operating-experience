Ubuntu 16.04 安装 tensorflow with GPU support（源码编译安装） 
-----------------------------------------


**显卡型号**  
NVIDIA Corporation GM204 [GeForce GTX 970]


##### 1. 安装依赖

* [安装CUDA 7.5](https://gist.github.com/dangbiao1991/2c895917ea888ce33af8c1c72444b7bf)

	安装完 CUDA 后需要配置环境变量

		$ sudo vim /etc/profile.d/cuda.sh

		export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64"
		export CUDA_HOME=/usr/local/cuda

		$ source  /etc/profile.d/cuda.sh

* 下载安装 cuDNN v5 [下载地址](https://developer.nvidia.com/cudnn)

		tar xvzf cudnn-7.5-linux-x64-v5.0-ga.tgz
		sudo cp cuda/include/cudnn.h /usr/local/cuda/include
		sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
		sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

* 安装 tensorflow 的编译工具 bazel [安装方法](http://bazel.io/docs/install.html)

* 安装其它依赖

		# For Python 2.7:
		$ sudo apt-get install python-numpy swig python-dev python-wheel
		# For Python 3.x:
		$ sudo apt-get install python3-numpy swig python3-dev python3-wheel


##### 2 .下载 tensorflow 源码，并配置编译选项
* 下载源码

		$ git clone --recurse-submodules https://github.com/tensorflow/tensorflow


* 配置编译选项

		$ cd tensorflow/

		$ ./configure
		Please specify the location of python. [Default is /usr/bin/python]:
		Do you wish to build TensorFlow with GPU support? [y/N] y
		GPU support will be enabled for TensorFlow

		Please specify which gcc nvcc should use as the host compiler. [Default is
		/usr/bin/gcc]: 
		Please specify the Cuda SDK version you want to use, e.g. 7.0. [Leave
		empty to use system default]: 7.5

		Please specify the location where CUDA 7.5 toolkit is installed. Refer to
		README.md for more details. [default is: /usr/local/cuda]: /usr/local/cuda

		Please specify the Cudnn version you want to use. [Leave empty to use system
		default]: 5.0.5

		Please specify the location where the cuDNN 5.0.5 library is installed. Refer to
		README.md for more details. [default is: /usr/local/cuda]:/usr/local/cuda

		Please specify a list of comma-separated Cuda compute capabilities you want to
		build with. You can find the compute capability of your device at:
		https://developer.nvidia.com/cuda-gpus.
		Please note that each additional compute capability significantly increases your
		build time and binary size. [Default is: \"3.5,5.2\"]: 5.2

		Setting up Cuda include
		Setting up Cuda lib64
		Setting up Cuda bin
		Setting up Cuda nvvm
		Setting up CUPTI include
		Setting up CUPTI lib64
		Configuration finished

##### 3. 编译 tensorflow 的 example_trainer 并运行

* GFW 问题修复。如果你使用国内的网络，编译过程中可能会遇到 https://boringssl.googlesource.com/boringssl 无法下载的问题。可以通过修改 workspace.bzl 将该源替换掉解决。

		$ sudo vim tensorflow/workspace.bzl

	并将以下内容

		native.new_git_repository(

		name = "boringssl_git",

		commit = "e72df93461c6d9d2b5698f10e16d3ab82f5adde3",

		remote = "https://boringssl.googlesource.com/boringssl",

		build_file = path_prefix + "boringssl.BUILD",

		)

	替换为


		native.new_git_repository(

		name = "boringssl_git",

		commit = “1eca1d3",

		remote = "https://github.com/google/boringssl.git",

		build_file = path_prefix + "boringssl.BUILD",

		)

* GCC 版本问题修复。目前Ubuntu 16.04 默认编译器版本为 GCC 5.3.1 ，使用此版本编译 tensorflow 可能会出现 memcpy 未定义的错误，可以通过添加编译选项 -D_FORCE_INLINES 解决。再下一步的编译 tensorflow 安装文件时也可能会遇到 builtin_ia32_mwaitx 未定义的错误，可以通过添加 编译选项 -D_MWAITXINTRIN_H_INCLUDED 和 -D__STRICT_ANSI__ 解决。具体修复方法如下


		$ vim third_party/gpus/crosstool/CROSSTOOL

	在该文件如下位置处

		toolchain {
		# Use "-std=c++11" for nvcc. For consistency, force both the host compiler
		# and the device compiler to use "-std=c++11".
		cxx_flag: "-std=c++11"
		linker_flag: "-lstdc++"
		linker_flag: "-B/usr/bin/"

	增加编译选项

		toolchain {
		# Use "-std=c++11" for nvcc. For consistency, force both the host compiler
		# and the device compiler to use "-std=c++11".
		cxx_flag: "-std=c++11"
		cxx_flag: "-D_FORCE_INLINES"
		cxx_flag: "-D_MWAITXINTRIN_H_INCLUDED"
		cxx_flag: "-D__STRICT_ANSI__"
		linker_flag: "-lstdc++"
		linker_flag: "-B/usr/bin/"


* 编译运行 example_trainer

		$ bazel build -c opt --config=cuda //tensorflow/cc:tutorials_example_trainer

		$ bazel-bin/tensorflow/cc/tutorials_example_trainer --use_gpu
		# Lots of output. This tutorial iteratively calculates the major eigenvalue of
		# a 2x2 matrix, on GPU. The last few lines look like this.
		000009/000005 lambda = 2.000000 x = [0.894427 -0.447214] y = [1.788854 -0.894427]
		000006/000001 lambda = 2.000000 x = [0.894427 -0.447214] y = [1.788854 -0.894427]
		000009/000009 lambda = 2.000000 x = [0.894427 -0.447214] y = [1.788854 -0.894427]


 	选项 --config=cuda 表示其支持 GPU

##### 4. 将 tensorflow 打包成 python wheel 文件并用 pip 安装

* 编译 build_pip_package

		# To build with GPU support:
		$ bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

* 打包到 pythone wheel

		$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

	打包的过程中可能会遇到找不到文件的错误提示：

		cp: cannot stat ‘bazel-bin/tensorflow/tools/pip_package/build_pip_package.runfiles/tensorflow’: No such file or directory
		cp: cannot stat ‘bazel-bin/tensorflow/tools/pip_package/build_pip_package.runfiles/external’: No such file or directory

	执行如下命令可以解决:(实际上是因为缺少的文件在 build_pip_package.runfiles/__main__/ 目录下)

		cp -r bazel-bin/tensorflow/tools/pip_package/build_pip_package.runfiles/__main__/* bazel-bin/tensorflow/tools/pip_package/build_pip_package.runfiles/



* 安装打包好的 whl 文件

		# The name of the .whl file will depend on your platform.
		$ sudo pip install /tmp/tensorflow_pkg/tensorflow-0.8.0-py2-none-any.whl


##### 5. 运行一个 sample model 测试安装结果

	$ cd ~/

	$ python -m tensorflow.models.image.mnist.convolutional
	Extracting data/train-images-idx3-ubyte.gz
	Extracting data/train-labels-idx1-ubyte.gz
	Extracting data/t10k-images-idx3-ubyte.gz
	Extracting data/t10k-labels-idx1-ubyte.gz
	...etc...


##### 参考


[1]https://github.com/tensorflow/tensorflow/blob/master/tensorflow/g3doc/get_started/os_setup.md


[2]https://github.com/tensorflow/tensorflow/issues/1066#issuecomment-200580370

[3]https://github.com/tensorflow/tensorflow/issues/1346

