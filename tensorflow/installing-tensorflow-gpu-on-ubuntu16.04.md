# Ubuntu16.04环境下安装TensorFlow-GPU版
## 环境
* 系统：Ubuntu
* 版本：16.04.3 LTS
* 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
* 内存：16.0GB
* 类型：64位操作系统 64位处理器
* 显卡：索泰GTX1060 6G
## 注意事项
* 显卡支持CUDA；
* 稳定的网络连接。
## 说明
* 本次安装的所有操作均是在root账户下执行的。可参照[opening-root-on-ubuntu](../ubuntu/opening-root-on-ubuntu.md)启用root用户。若是要在非root账户下执行，请注意修改命令中的路径。
## 软件下载
* CUDA8.0：[官网下载](http://developer2.download.nvidia.com/compute/cuda/8.0/secure/Prod2/local_installers/cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb)， [百度网盘下载](https://pan.baidu.com/s/1c1BTZW0)；
* cuDNN6.0：[百度网盘下载](https://pan.baidu.com/s/1o78RO6m)；
## 安装流程
### 安装显卡驱动
1. 使用命令`vim /etc/modprobe.d/blacklist.conf`编辑文件，在文件末尾输入以下内容以屏蔽系统集成显卡驱动：
  ```text
  blacklist vga16fb
  blacklist nouveau
  blacklist rivafb
  blacklist rivatv
  blacklist nvidiafb
  ```
2. 使用快捷键`Ctrl+Alt+F1`切换至命令行界面；
3. 关闭图形系统：`service lightdm stop`；
4. 删除之前的驱动：`apt purge nvidia-*`；
5. 更新源：`apt update`；
6. 查询驱动的可用版本：`apt-cache search nvidia-*`；
7. 安装驱动：`apt install nvidia-xxx`；**xxx是上一步中查询到可用的最新版本号。**
8. 输入`reboot`命令重启系统。
### 安装CUDA8.0
1. 下载文件存放于`/root/Download/`目录下；
2. 执行以下命令进行安装：
  ```console
  $ dpkg -i cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb
  $ apt update
  $ apt install cuda
  ```
3. 输入命令`vim /root/.bashrc`编辑文件以配置环境变量，在打开的文件末尾追加以下内容：
  ```text
  export CUDA_HOME=/usr/local/cuda
  export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$CUDA_HOME/extras/CUPTI/lib64:$LD_LIBRARY_PATH
  export PATH=$CUDA_HOME/bin:$PATH
  ```
4. 输入命令`source /root/.bashrc`使环境变量生效。
5. 输入命令`nvcc -V`查看版本号以验证CUDA是否安装成功。
### 安装cuDNN6.0
1. 下载文件存放于`/root/Download/`目录下；
2. 执行一下命令完成安装：
  ```console
  $ cd /root/Download/
  $ tar -zxvf ./cudnn-8.0-linux-x64-v6.0.tgz -C $CUDA_HOME/../
  ```
### 安装Tensorflow-GPU
1. 安装pip：`apt install python-pip`
2. 国内用户请参照[configuring-pip-to-use-domestic-source](https://github.com/hemajun815/tutorial/blob/master/pip/1.configuring-pip-to-use-domestic-source.md)将pip配置成使用国内源，可大幅度提升安装时的下载速度；
3. 更新pip：`pip install -U pip`
4. 安装tensorlfow-gpu：`pip install tensorflow-gpu`
## 验证安装
1. 启动python命令行：`python`；
2. 引入tensorflow：`import tensorflow as tf`；
3. 查看tensorflow版本：`tf.__version__`；
4. 创建Session：`tf.Session()`，可以在设备名处看到你的显卡名称表示安装完成；
5. 退出python：`exit()`。
## 附fix
* **问题**：【20170925】`/sbin/ldconfig.real: /usr/lib/nvidia-xxx/libEGL.so.1 is not a symbolic link`
* **解决方案**：
  1. 下载[fix-libegl-so-1.sh](https://pan.baidu.com/s/1nvMHp1n)；
  2. 添加执行权限：`chmod +x ./fix-libegl-so-1.sh`；
  3. 执行脚本：`./fix-libegl-so-1.sh`，过程中输入`yes`。
