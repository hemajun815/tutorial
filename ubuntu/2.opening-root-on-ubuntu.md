# 启用Ubuntu16.04系统的root用户
1. 设置root用户密码：
	```console
	$ sudo passwd root
	```
2. 配置允许root用户登录：使用命令`sudo vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf`编辑文件，在文件最后增加`greeter-show-manual-login=true`，保存文件，退出。
3. 关闭guest用户登录：使用命令`sudo vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf`编辑文件，在文件最后增加`allow-guest=false`，保存文件，退出。
4. 重启系统，登录界面即可选择root用户登录。
5. 使用root登录后，会出现以下问题：
	```text
	Error found when loading /root/.profile:
	mesg: ttyname failed: Inappropriate ioctl for device
	As a result the session will not be configured correctly.
	You should fix the problem as soon as feasible.
	```
6. 点击OK按钮进入系统，打开终端，输入命令`vim /root/.profile`编辑文件，注释或者删除掉`mesg n || true`，在文件尾追加`tty -s && mesg n`，保存文件，退出，重启系统。
7. 至此，root用户启用成功。
