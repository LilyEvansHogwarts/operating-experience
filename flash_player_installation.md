# Flash Player Installation

## 1. 进入flash player官方页面下载.tar.gz包

## 2. 解压flash player的tar.gz的文件

	tar -zxvf flash_player_XXXXXX.tar.gz
	最终得到的文件为:
	readme.txt  libflashplayer.so  LGPL/  usr/ license.pdf

## 3. installation

	sudo cp libflashplayer.so **brower-plugins-location**
	对于firefox而言，**brower-plugins-location**为/usr/lib/firefox-addons/plugins/

	sudo cp -r usr/* /usr
	将/usr下的所有文件都移至root/usr/下

	重启firefox
