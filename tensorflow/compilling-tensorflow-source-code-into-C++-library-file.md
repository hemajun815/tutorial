# 将Tensorflow源码编译成C++库文件

## 环境

- 系统：Ubuntu
- 版本：16.04.3 LTS
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器
- 显卡：索泰GTX1060 6G

## 步骤

### 安装Bazel

1. 安装JDK8：`sudo apt install openjdk-8-jdk`
2. 添加Bazel分发源地址：
   ```
   echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
   curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
   ```
3. 安装Bazel：`sudo apt update && sudo apt install bazel`
4. 更新Bazel：`sudo apt upgrade bazel`

### 安装Protobuf

1. 安装automake和libtool：`sudo apt install automake libtool`
2. 从GitHub上下载Protobuf源码：`git clone https://github.com/google/protobuf.git`
3. 进入Protobuf源码：`cd protobuf`
4. 运行脚本：`./autogen.sh`
5. 配置：`./configure`
6. 检查环境：`make check`
7. 编译：`make -j8`
8. 安装：`sudo make install`
9. 添加环境变量：`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib`
10. 查看版本以验证安装：`protoc --version`

### 安装Eigen

1. `sudo apt install libeigen3-dev`

### 编译TensorFlow

1. 从GitHub上下载TensorFlow源码：`git clone --recursive https://github.com/tensorflow/tensorflow`
2. 进入TensorFlow根目录：`cd tensorflow`
3. 使用Bazel编译C++ API的库：`bazel build //tensorflow:libtensorflow_cc.so`


### 安装TensorFlow C++库

1. 建立TensorFlow库文件夹：`sudo mkdir /usr/local/tensorflow`
2. 复制include文件：
   ```
   sudo mkdir /usr/local/tensorflow/include
   sudo cp -r bazel-genfiles/ /usr/local/tensorflow/include/
   sudo cp -r tensorflow /usr/local/tensorflow/include/
   sudo cp -r third_party /usr/local/tensorflow/include/
   ```
3. 复制lib文件：
   ```
   sudo mkdir /usr/local/tensorflow/lib
   sudo cp -r bazel-bin/tensorflow/libtensorflow_cc.so /usr/local/tensorflow/lib/
   ```
