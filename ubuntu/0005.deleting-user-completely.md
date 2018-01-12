# 彻底地删除用户
## 环境
- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS
## 步骤
1. 启动终端：`ctrl+alt+t`；
2. root权限下执行命令`userdel -r xxx`，非root权限执行`sudo userdel -r xxx`（其中xxx是要删除的用户名）。
## 说明
此方式删除用户的同时会一起把这个用户的宿主目录和邮件目录删除。
