# Ubuntu默认启动到命令行
## 环境
- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS
- 用户权限：root（启用root用户可参考[启用Ubuntu16.04系统的root用户](./opening-root-on-ubuntu.md)进行操作）
## 准备工作
- 使用命令`apt install vim`安装好vim软件。
## 步骤
1. 使用命令`vim /etc/default/grub`编辑grub；
2. 按`i`进入vim编辑模式；
3. 注释掉“GRUB_CMDLINE_LINUX_DEFAULT="quiet"”，修改“GRUB_CMDLINE_LINUX=""”为“GRUB_CMDLINE_LINUX="text"”，取消“#GRUB_TERMINAL=console”的注释；
4. 按`esc`退出vim编辑模式；
5. 按`:`进入vim命令模式，输入`wq`命令保存文件并退出vim；
6. 使用命令`update-grub`更新grub；
7. 使用命令`reboot`重启系统。
## 附fix
- **问题**：【20171005】以上方式对新版本Ubuntu无效。
- **解决方案**：
  - 终端执行命令`systemctl set-default multi-user.target`默认启动到命令行界面。
  - 终端执行命令`systemctl set-default graphical.target`默认启动到图形界面。
  - 命令行节目启动图形界面执行命令`systemctl start lightdm`。
