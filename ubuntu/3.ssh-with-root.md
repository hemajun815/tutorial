# 允许使用root用户登录ssh服务
## 环境
- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS
## 前言
启用root用户请依照[这里](https://github.com/hemajun815/tutorial/blob/master/ubuntu/2.opening-root-on-ubuntu.md)进行操作。
## 准备工作
- 使用命令`sudo apt install openssh-server`安装好ssh服务。
- 使用命令`sudo apt install vim`安装好vim软件。
## 配置步骤
1. 编辑ssh的配置文件：`sudo vim /etc/ssh/sshd_config`；
2. 按`i`进入vim编辑模式；
3. 在Authentication部分，注释掉“PermitRootLogin prohibit-password”，并添加“PermitRootLogin yes”；
4. 按`esc`退出vim编辑模式；
5. 按`:`进入vim命令模式，输入`wq`命令保存文件并退出vim；
6. 输入命令`sudo service ssh restart`重启ssh服务。
