# Ubuntu16.04环境下基于python2.7安装pyqt5

## 环境

- 系统：Ubuntu
- 版本：16.04.3 LTS
- python环境：python2.7

## 软件下载

- QT5.9.3：[官网下载](http://download.qt.io/archive/qt/5.9/5.9.3/qt-opensource-linux-x64-5.9.3.run)，[百度网盘下载](https://pan.baidu.com/s/1pMcySpd)
- SIP4.19.6：[官网下载](https://ncu.dl.sourceforge.net/project/pyqt/sip/sip-4.19.6/sip-4.19.6.tar.gz)，[百度网盘下载](https://pan.baidu.com/s/1nxiiQhV)
- PyQt5：[官网下载](https://ncu.dl.sourceforge.net/project/pyqt/PyQt5/PyQt-5.9.2/PyQt5_gpl-5.9.2.tar.gz)，[百度网盘下载](https://pan.baidu.com/s/1eSPPZjW)

## 安装流程

### 安装QT

1. 下载软件：`wget http://download.qt.io/archive/qt/5.9/5.9.3/qt-opensource-linux-x64-5.9.3.run`
2. 添加执行权限：`chmod +x qt-opensource-linux-x64-5.9.3.run`
3. 启动安装器：`sudo bash qt-opensource-linux-x64-5.9.3.run`
4. 此时，会弹出图形界面安装器，跟随提示一步一步安装完毕。

### 安装SIP

1. 下载软件：`wget https://ncu.dl.sourceforge.net/project/pyqt/sip/sip-4.19.6/sip-4.19.6.tar.gz`
2. 解压安装包：`tar -zxvf sip-4.19.6.tar.gz -C .`
3. 进入安装包：`cd sip`
4. 配置：`python configure.py`
5. 编译：`make -j10`
6. 安装：`sudo make install`
7. 输入`sip -V`命令可查看sip版本号。

### 安装PyQt5

1. 下载软件：`wget https://ncu.dl.sourceforge.net/project/pyqt/PyQt5/PyQt-5.9.2/PyQt5_gpl-5.9.2.tar.gz`

2. 解压安装包：`tar -zxvf PyQt5_gpl-5.9.2.tar.gz -C .`

3. 进入安装包：`cd PyQt5`

4. 配置：`python configure.py --qmake ${QtHome}/5.9.3/gcc_64/bin/qmake`（其中请把`${QtHome}`替换为Qt的安装目录）

5. 编译：`make -j10`

6. 安装：`sudo make install`

7. 至此PyQt5安装完成，测试实例如下：

   ```python
   # -*- coding: utf-8 -*-
   from PyQt5 import QtWidgets
   import sys
   	app = QtWidgets.QApplication(sys.argv)
   	first_window = QtWidgets.QWidget()
   	first_window.resize(400, 300)
   	first_window.setWindowTitle("我的第一个程序")
   	first_window.show()
   	sys.exit(app.exec_())
   ```

   ​