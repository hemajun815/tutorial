# 安装Ubuntu16.04后的配置工作
## 建议
1. 删除libreoffice、amazon的链接、不用的自带软件
	```console
	$ sudo apt-get remove libreoffice-common
	$ sudo apt-get remove unity-webapps-common
	$ sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku landscape-client-ui-install
	$ sudo apt-get remove onboard deja-dup
	```
2. 卸载火狐浏览器
	```console
	$ sudo apt-get purge *firefox*
	```
3. 卸载ubuntu软件中心
	```console
	$ sudo apt-get purge gnome-software
	```
4. 系统升级
	```console
	$ sudo apt-get update
	$ sudo apt-get upgrade
	$ sudo apt-get dist-upgrade
	```
## 可选
5. 安装谷歌浏览器
	```console
	$ sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
	$ wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
	$ sudo apt-get update
	$ sudo apt-get install google-chrome-stable
	```
6. 安装Sublime Text 3	
	```console
	$ sudo add-apt-repository ppa:webupd8team/sublime-text-3 
	$ sudo apt-get update  
	$ sudo apt-get install sublime-text
	```
7. 自动清理
	```console
	$ sudo apt autoremove
	```
8. [启用root用户](https://github.com/hemajun815/tutorial/blob/master/ubuntu/opening-root-on-ubuntu.md)
9. [配置pip使用国内源](https://github.com/hemajun815/tutorial/blob/master/pip/configuring-pip-to-use-domestic-source.md)
