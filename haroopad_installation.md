# Haroopad Installation

## 1. download haroopad  tar.gz package file

[haroopad download page](https://bitbucket.org/rhiokim/haroopad-download/downloads/haroopad-v0.13.1-x64.tar.gz)

> $ wget https://bitbucket.org/rhiokim/haroopad-download/downloads/haroopad-v0.13.1-x64.tar.gz
> ###### 由于haroopad下载网站需要翻墙下载才比较快所以可以在命令之前加上`proxychains`

## 2. install haroopad

* cd ~/opt/haroopad-v0.13.1-x64.tar.gz
 
	mkdir haroopad
	tar zxvf haroopad-v0.13.1-x64.tar.gz -C haroopad
	cd haroopad
	tar zxvf data.tar.gz
	sudo cp -r ./usr /

	tar zxf control.tar.gz
	chmod 755 postinst
	sudo ./postinst

## 3. repair haroopad desktop icon

> $ sudo vi /usr/share/applications/Haroopad.desktop
> ###### replace icon value to /usr/share/icons/hicolor/128x128/apps/haroopad.png

* finally haroopad.desktop:

	[Desktop Entry]
	Name=haroopad
	Version=0.13.1
	Exec=haroopad
	Comment=The Next Document processor based on Markdown
	Icon=/usr/share/icons/hicolor/128x128/apps/haroopad.png
	Type=Application
	Terminal=false
	StartupNotify=true
	Encoding=UTF-8
	Categories=Development;GTK;GNOME;

