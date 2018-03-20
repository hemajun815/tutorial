# 安装 Ubuntu16.04 后的配置工作
## 建议
1. 删除libreoffice、amazon的链接、不用的自带软件
  ```console
  $ sudo apt purge libreoffice-common
  $ sudo apt purge unity-webapps-common
  $ sudo apt purge thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku landscape-client-ui-install
  $ sudo apt purge onboard deja-dup
  ```
2. 卸载火狐浏览器
  ```console
  $ sudo apt purge *firefox*
  ```
3. 卸载ubuntu软件中心
  ```console
  $ sudo apt purge gnome-software
  ```
4. 系统升级
  ```console
  $ sudo apt update
  $ sudo apt upgrade
  ```
## 可选
5. 安装谷歌浏览器

  ```console
  $ sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
  $ wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
  $ sudo apt update
  $ sudo apt install google-chrome-stable
  ```
  
6. 安装Sublime Text 3

  ```console
  $ sudo add-apt-repository ppa:webupd8team/sublime-text-3 
  $ sudo apt update  
  $ sudo apt install sublime-text
  ```
  
  > 更多有关 Sublime 的教程请点击[这里](../sublime/readme.md)

7. 自动清理

  ```console
  $ sudo apt autoremove
  ```

8. [更换Flatabulous主题](./installing-flat-theme:Flatabulous.md)

9. [启用root用户](./opening-root-on-ubuntu.md)

10. [配置pip使用国内源](../pip/configuring-pip-to-use-domestic-source.md)
