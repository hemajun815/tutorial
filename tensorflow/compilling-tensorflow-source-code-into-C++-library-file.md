# 将Tensorflow源码编译成C++库文件

## 环境

- 系统：Ubuntu
- 版本：16.04.3 LTS
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器
- 显卡：索泰GTX1060 6G

## 步骤

这里提供两种安装方式：

- [方式一](#方式一)自动安装第三方依赖，步骤简单，自动从Github下载，速度较慢。
- [方式二](#方式二)手动安装第三方依赖，步骤详尽，手动下载安装，速度较快。

### 方式一

#### 安装Bazel

1. 安装JDK8：`sudo apt install openjdk-8-jdk`

2. 添加Bazel分发源地址：

   ```
   echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
   curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
   ```

3. 安装Bazel：`sudo apt update && sudo apt install bazel`

4. 更新Bazel：`sudo apt upgrade bazel`

#### 配置第三方依赖

1. 从GitHub上下载TensorFlow源码：`git clone --recursive https://github.com/tensorflow/tensorflow`
2. 进入目录：`cd tensorflow/contrib/makefile`
3. 执行文件：`./build_all_linux.sh`

#### 编译TensorFlow

1. 进入TensorFlow根目录：`cd tensorflow`
2. 配置TensorFlow：`./configure`
3. 使用Bazel编译C++ API的库：`bazel build //tensorflow:libtensorflow_cc.so`

#### 安装

1. 建立文件夹

```
sudo mkdir /usr/local/tensorflow
```

2. 拷贝头文件

```
sudo mkdir /usr/local/tensorflow/include
sudo cp -r tensorflow/contrib/makefile/downloads/eigen/Eigen /usr/local/tensorflow/include/
sudo cp -r tensorflow/contrib/makefile/downloads/eigen/unsupported /usr/local/tensorflow/include/
sudo cp -r tensorflow/contrib/makefile/gen/protobuf/include/google /usr/local/tensorflow/include/
sudo cp tensorflow/contrib/makefile/downloads/nsync/public/* /usr/local/tensorflow/include/
sudo cp -r bazel-genfiles/tensorflow /usr/local/tensorflow/include/
sudo cp -r tensorflow/cc /usr/local/tensorflow/include/tensorflow
sudo cp -r tensorflow/core /usr/local/tensorflow/include/tensorflow
sudo mkdir /usr/local/tensorflow/include/third_party
sudo cp -r third_party/eigen3 /usr/local/tensorflow/include/third_party/
```

3. 拷贝库文件

```
sudo mkdir /usr/local/tensorflow/lib
sudo cp bazel-bin/tensorflow/libtensorflow_*.so /usr/local/tensorflow/lib
```

---

### 方式二

#### 安装Bazel

1. 安装JDK8：`sudo apt install openjdk-8-jdk`

2. 添加Bazel分发源地址：
   ```
   echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
   curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
   ```

3. 安装Bazel：`sudo apt update && sudo apt install bazel`

4. 更新Bazel：`sudo apt upgrade bazel`

#### 安装Protobuf

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

#### 安装Eigen

1. `sudo apt install libeigen3-dev`

#### 编译TensorFlow

1. 从GitHub上下载TensorFlow源码：`git clone --recursive https://github.com/tensorflow/tensorflow`
2. 进入TensorFlow根目录：`cd tensorflow`
3. 配置TensorFlow：`./configure`
4. 使用Bazel编译C++ API的库：`bazel build //tensorflow:libtensorflow_cc.so`


#### 安装

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

---

## 调用

不论是使用了[方式一](#方式一)安装，还是使用了[方式二](#方式二)安装，至此，我们已经成功将Tensorflow源码编译成了C++库文件，附上`g++`命令编译方式：

```console
g++ -std=c++11 ./main.cc -ltensorflow_framework -ltensorflow_cc -o tfcc \
-I/usr/local/tensorflow/include/ \
-L/usr/local/tensorflow/lib/ \
-Wl,-rpath=/usr/local/tensorflow/lib/
```

更多使用方式，可参考教程：[仅用TensorFlow C++训练一个DNN](./training-a-DNN-using-only-tensorflow-cc.md)

## 附 Fix

- **问题**：The TensorFlow library wasn't compiled to use *SSE4.1/SSE4.2/FMA/AVX/AVX2* instructions, but these are available on your machine and could speed up CPU computations.
- **解决方案**：bazel 编译源码时加上选项后执行编译：`bazel build -c opt --copt=-mavx --copt=-mfma --copt=-mavx2 --copt=-msse4.1 --copt=-msse4.2 -k //tensorflow:libtensorflow_cc.so`