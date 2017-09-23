# TensorFlow入门
## 环境
* Ubuntu 16.04.2
* Python 2.7
* TensorFlow 1.3 rc0
## TensorFlow简介
TensorFlow是一个采用数据流图（data flow graphs），用于数值计算的开源软件库。它灵活的架构让你可以在多种平台上展开计算，例如台式计算机中的一个或多个CPU（或GPU），服务器，移动设备等等。那什么是数据流图呢？数据流图用“结点”（nodes）和“线”(edges)的有向图来描述数学计算。“节点” 一般用来表示施加的数学操作，但也可以表示数据输入的起点/输出的终点，或者是读取/写入持久变量的终点。“线”表示“节点”之间的输入/输出关系。这些数据“线”可以输运“size可动态调整”的多维数据数组，即“张量”（tensor）。TensorFlow具有灵活性高、移植性强、支持语言多、性能最优化的特征。
## TensorFlow安装
### 源码安装
1. 安装git：
```console
$ apt-get install git
```
2. 下载TensorFlow源码：
```console
$ git clone --recurse-submodules https://github.com/tensorflow/tensorflow
```
3. 安装JDK8
```console
$ apt-get install openjdk-8-jdk
```
4. 安装Bazel
```console
$ echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
$ curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
$ apt-get update
$ apt-get install bazel
```
5. 安装其他依赖
```console
$ apt-get install python-numpy swig python-dev
```
6. 运行TensorFlow根目录下的configure脚本：
```console
$ cd tensorflow
$ ./configure
```
7. 编译目标程序(示例)：
```console
$ bazel build -c opt //tensorflow/cc:tutorials_example_trainer
```
8. 运行目标程序(示例)：
```console
$ bazel-bin/tensorflow/cc/tutorials_example_trainer
```
### pip安装
1. 安装python和pip：
```console
$ apt-get install python-pip python-dev   # for Python 2.7
$ apt-get install python3-pip python3-dev # for Python 3.n
```
2. 安装TensorFlow：
```console
$ pip install tensorflow      # Python 2.7; CPU support (no GPU support)
$ pip3 install tensorflow     # Python 3.n; CPU support (no GPU support)
$ pip install tensorflow-gpu  # Python 2.7;  GPU support
$ pip3 install tensorflow-gpu # Python 3.n; GPU support 
```
3. (附)卸载TensorFlow：
```console
$ pip uninstall tensorflow  # for Python 2.7
$ pip3 uninstall tensorflow # for Python 3.n
```
### 验证安装
* 启动python：
```console
$ python
```
* 输入：
```console
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> sess.run(hello)
```
* 若是成功安装了TensorFlow，则会输出：
```console
Hello, TensorFlow!
```
## TensorFlow核心
### Tensor
* 在数学上，Matrix表示二维线性映射，Tensor表示多维线性映射，Tensor是对Matrix的泛化，可以表示1-dim、2-dim、N-dim的高维空间。
* Tensor在高维空间数学运算比Matrix计算复杂，计算量也非常大，加速张量并行运算是TF优先考虑的问题，如add, contract, slice, reshape, reduce, shuffle等运算。
* TensorFlow中Tensor的维数描述为阶，数值是0阶，向量是1阶，矩阵是2阶，以此类推，可以表示n阶高维数据。
* TensorFlow中Tensor支持的数据类型有很多，如tf.float16, tf.float32, tf.float64, tf.uint8, tf.int8, tf.int16, tf.int32, tf.int64, tf.string, tf.bool, tf.complex64等，所有Tensor运算都使用泛化的数据类型表示。
### 符号式编程
* TensorFlow采用的是符号式编程，不同于常见的命令式编程，符号式编程是将计算过程抽象为计算图，计算流图可以方便的描述计算过程，所有输入节点、运算节点、输出节点均符号化处理。
* 计算图通过建立输入节点到输出节点的传递闭包，从输入节点出发，沿着传递闭包完成数值计算和数据流动，直到达到输出节点。这个过程经过计算图优化，以数据流方式完成，节省内存空间使用，计算速度快，但不适合程序调试，通常不用于编程语言中。
## TensorFlow编程（Python API）
### Hello, TensorFlow
第一个TensorFlow程序
* 代码：
```python
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow.')
sess = tf.Session()
print sess.run(hello)
```
* 输出：
```console
Hello, TensorFlow.
```
### 常量
通过`tf.constant`创建一个常量，然后启动`tf.Session()`，调用`tf.Session()`的`run()`方法来启动整个数据图完成操作。
* 代码：
```python
import tensorflow as tf
a = tf.constant(2)
b = tf.constant(3)
with tf.Session() as sess:
	print 'a = %i, b = %i' % (sess.run(a), sess.run(b))
	print 'Addition with constants: %i' % sess.run(a + b)
	print 'Multiplication with constants: %i' % sess.run(a * b)
```
* 输出：
```console
a = 2, b = 3
Addition with constants: 5
Multiplication with constants: 6
```
### 变量
* 代码：
```python
import tensorflow as tf
a = tf.placeholder(tf.int16)
b = tf.placeholder(tf.int16)
add = tf.add(a, b)
mul = tf.multiply(a, b)
with tf.Session() as sess:
	print 'Addition with variables: %i' % sess.run(add, feed_dict={a: 2, b: 3})
	print 'Multiplication with variables: %i' % sess.run(mul, feed_dict={a: 2, b: 3})
```
* 输出：
```console
Addition with variables: 5
Multiplication with variables: 6
```
### 线性回归
* 代码：
```python
import tensorflow as tf
import numpy

# Parameters
learning_rate = 0.01
training_epochs = 2000
display_step = 50

# Train data
train_X = numpy.asarray([3.3,4.4,5.5,6.71,6.93,4.168,9.779,6.182,7.59,2.167,7.042,10.791,5.313,7.997,5.654,9.27,3.1])
train_Y = numpy.asarray([1.7,2.76,2.09,3.19,1.694,1.573,3.366,2.596,2.53,1.221,2.827,3.465,1.65,2.904,2.42,2.94,1.3])
n_samples = train_X.shape[0]

# TensorFlow graph input
X = tf.placeholder(tf.float32)
Y = tf.placeholder(tf.float32)

# Random model weight
W = tf.Variable(numpy.random.randn())
b = tf.Variable(numpy.random.randn())

# Construct a linear model
activation = tf.add(tf.multiply(X, W), b)

# Minimize the squared errors
cost = tf.reduce_sum(tf.pow(activation - Y, 2)) / (2 * n_samples) #L2 loss
optimizer = tf.train.GradientDescentOptimizer(learning_rate).minimize(cost) #Gradient descent

# Lunch the graph
with tf.Session() as sess:
	sess.run(tf.global_variables_initializer());
	for epoch in range(training_epochs):
		for (x, y) in zip(train_X, train_Y):
			sess.run(optimizer, feed_dict = {X: x, Y: y})
		if epoch % display_step == 0:
			print "Epoch:", '%04d' % (epoch+1), "cost=", "{:.9f}".format(sess.run(cost, feed_dict={X: train_X, Y:train_Y})), "W=", sess.run(W), "b=", sess.run(b)
	print "Optimization Finished!"
	print "cost=", sess.run(cost, feed_dict={X: train_X, Y: train_Y}), "W=", sess.run(W), "b=", sess.run(b)
```
* 输出：
```console
Epoch: 0001 cost= 30.090188980 W= -0.786318 b= -0.0924709
Epoch: 0051 cost= 0.106197909 W= 0.345267 b= 0.113195
Epoch: 0101 cost= 0.102819055 W= 0.339577 b= 0.15413
Epoch: 0151 cost= 0.099830635 W= 0.334226 b= 0.19263
Epoch: 0201 cost= 0.097187586 W= 0.329192 b= 0.228841
Epoch: 0251 cost= 0.094849996 W= 0.324458 b= 0.262897
Epoch: 0301 cost= 0.092782550 W= 0.320005 b= 0.294929
Epoch: 0351 cost= 0.090954095 W= 0.315818 b= 0.325055
Epoch: 0401 cost= 0.089337043 W= 0.311879 b= 0.353389
Epoch: 0451 cost= 0.087906934 W= 0.308175 b= 0.380039
Epoch: 0501 cost= 0.086642161 W= 0.30469 b= 0.405104
Epoch: 0551 cost= 0.085523665 W= 0.301413 b= 0.428677
Epoch: 0601 cost= 0.084534548 W= 0.298332 b= 0.450849
Epoch: 0651 cost= 0.083659820 W= 0.295433 b= 0.471702
Epoch: 0701 cost= 0.082886301 W= 0.292706 b= 0.491315
Epoch: 0751 cost= 0.082202256 W= 0.290142 b= 0.509761
Epoch: 0801 cost= 0.081597380 W= 0.287731 b= 0.527111
Epoch: 0851 cost= 0.081062540 W= 0.285462 b= 0.543428
Epoch: 0901 cost= 0.080589607 W= 0.283329 b= 0.558776
Epoch: 0951 cost= 0.080171384 W= 0.281322 b= 0.573211
Epoch: 1001 cost= 0.079801634 W= 0.279435 b= 0.586787
Epoch: 1051 cost= 0.079474717 W= 0.27766 b= 0.599556
Epoch: 1101 cost= 0.079185642 W= 0.275991 b= 0.611566
Epoch: 1151 cost= 0.078930154 W= 0.274421 b= 0.62286
Epoch: 1201 cost= 0.078704290 W= 0.272945 b= 0.633481
Epoch: 1251 cost= 0.078504585 W= 0.271556 b= 0.643471
Epoch: 1301 cost= 0.078328066 W= 0.27025 b= 0.652867
Epoch: 1351 cost= 0.078172021 W= 0.269021 b= 0.661704
Epoch: 1401 cost= 0.078034066 W= 0.267866 b= 0.670015
Epoch: 1451 cost= 0.077912129 W= 0.266779 b= 0.677833
Epoch: 1501 cost= 0.077804357 W= 0.265757 b= 0.685186
Epoch: 1551 cost= 0.077709131 W= 0.264796 b= 0.692102
Epoch: 1601 cost= 0.077624954 W= 0.263892 b= 0.698606
Epoch: 1651 cost= 0.077550553 W= 0.263041 b= 0.704724
Epoch: 1701 cost= 0.077484816 W= 0.262241 b= 0.710478
Epoch: 1751 cost= 0.077426739 W= 0.261489 b= 0.71589
Epoch: 1801 cost= 0.077375442 W= 0.260782 b= 0.72098
Epoch: 1851 cost= 0.077330098 W= 0.260116 b= 0.725768
Epoch: 1901 cost= 0.077290058 W= 0.25949 b= 0.73027
Epoch: 1951 cost= 0.077254690 W= 0.258902 b= 0.734505
Optimization Finished!
cost= 0.077224 W= 0.258359 b= 0.738412
```
