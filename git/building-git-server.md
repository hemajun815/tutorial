# 搭建 git 服务器

1. 安装 `git`：

   `sudo apt install git`

2. 创建用户，用以运行 `git`服务：

   `sudo adduser git`

3. 收集需要登录的用户的公钥，加入到`/home/git/.ssh/authorized_keys`文件：

   `sudo cat id_rsa.pub > /home/git/.ssh/authorized_keys`

4. 创建一个目录作为 `git` 根目录：

   `sudo mkdir /git-db`

5. 初始化 `git` 仓库：

   `cd /git-db && git init --bare test.git`

6. 将新仓库的 owner 设置为 git 用户：

   `sudo chown -R git:git ./test.git`

7. 禁用 git 用户的 shell 登录：

   `sudo vim /etc/passwd`

   将 

   `git:x:1001:1001:,,,:/home/git:/bin/bash` 

   改为 

   `git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell` 

8. git 服务器搭建完成，可以远程克隆了：

   `git clone git@server:/git-db/test.git`