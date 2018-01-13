# TensorFlow多机部署
## 环境
* Ubuntu 16.04.2
* Python 2.7
* TensorFlow 1.3 rc0
## Local Server
#### 简介
TensorFlow从0.8版本开始，支持分布式集群，并且自带了Local Server方便测试。Local Server与分布式服务的接口一致，且简单易操作，先弄明白Local Server的使用对后面的研究是有很大帮助的。
#### 入门操作
安装好TensorFlow之后，在终端输入以下命令即可启动并调用Local Server：
```console
$ python
>>> import tensorflow as tf
>>> server=tf.train.Server.create_local_server()
>>> h=tf.constant('hello')
>>> s=tf.Session(server.target)
>>> s.run(h)
>>> s.close()
```
注意这里的服务端Server和客户端Client都是启动在同一进程中，我们都知道TensorFlow客户端可以用Python或C++编写，本质上就是构建一个可执行的Graph，而Graph需要在Session中执行，因此代码中只要包含Session()的就是客户端。在这里，正是通过server.target选择本地刚创建的server来执行op的。
#### 服务端
前面提到Server和Client在同一进程中，程序执行完了就都退出了，但在实际中，Server应一直存在，等待Client的请求。编写Server代码server.py如下：
```python
import tensorflow as tf
server=tf.train.Server.create_local_server()
server.join()
```
注意这里的`join()`操作是为了保证Server不退出。通过以下命令启动：
```console
$ python server.py
```
启动之后服务端会暴露一个grpc端口供客户端调用：
```console
2017-09-09 09:40:11.242626: I tensorflow/core/distributed_runtime/rpc/grpc_channel.cc:215] Initialize GrpcChannelCache for job local -> {0 -> localhost:33902}
2017-09-09 09:40:11.245950: I tensorflow/core/distributed_runtime/rpc/grpc_server_lib.cc:316] Started server with target: grpc://localhost:33902
```
#### 客户端
我们可以通过服务端暴露的端口（grpc://localhost:33902）编写客户端代码client.py请求服务：
```python
import tensorflow as tf
h = tf.constant('hello')
with tf.Session('grpc://localhost:33902') as s:
	s.run(h)
```
在另一进程中调用客户端代码：
```console
$ python client.py
```
大家会觉得这部分代码与单机版TensorFlow的例子很相像，但是注意这里的`Session()`参数不同，这个op实际上是在Server进程中执行完之后返回的。
## 多线程实践
#### 简介
分布式TensorFlow集群由多个Server进程和Client进程组成，在某些场景下，服务端和客户端可以写到同一个Python文件并起在同一个进程，但为了更好理解分布式架构，我们将启动多个进程进行实践。
#### 集群配置
在本次实践中，集群中设置了一台参数服务器和两台工作服务器。如下所示：
```python
cluster_spec = { 'ps': ['127.0.0.1:2221'], 'worker': ['127.0.0.1:2222', '127.0.0.1:2223'] }
```
#### 服务端
根据集群配置，启动集群中的服务器，编写代码server.py如下：
```python
import sys
import argparse
import tensorflow as tf

def parse_arguments(argv):
	parser = argparse.ArgumentParser()
	parser.add_argument('job_name', type=str, help='Job name.')
	parser.add_argument('task_index', type=int, help='Task index.')
	return parser.parse_args(argv)

def main(args):
	cluster_spec = { 'ps': ['127.0.0.1:2221'], 'worker': ['127.0.0.1:2222', '127.0.0.1:2223'] }
	cluster = tf.train.ClusterSpec(cluster_spec)
	tf.train.Server(cluster, job_name=args.job_name, task_index=args.task_index).join()

if __name__ == '__main__':
	main(parse_arguments(sys.argv[1:]))
```
通过不同的参数调用，启动所有的服务器：
```console
$ python server.py ps 0 &
$ python server.py worker 0 &
$ python server.py worker 1 &
```
#### 客户端
编写客户端代码client.py调用服务器完成线性规划：
```python
import sys
import argparse
import tensorflow as tf
import numpy as np

def parse_arguments(argv):
	parser = argparse.ArgumentParser()
	parser.add_argument('server_port', type=str, help='Server port.')
	return parser.parse_args(argv)

def main(args):
	cluster_spec = { 'ps': ['127.0.0.1:3001'], 'worker': ['127.0.0.1:3002', '127.0.0.1:3003'] }
	cluster = tf.train.ClusterSpec(cluster_spec)

	train_X = np.linspace(-1, 1, 101)
	train_Y = 2 * train_X + np.random.randn(*train_X.shape) * 0.33 + 10

	max_epochs = 1000
	learning_rate = 0.01
	checkpoint_period = 50

	X = tf.placeholder(tf.float32)
	Y = tf.placeholder(tf.float32)
	w = tf.Variable(0., name = 'weight')
	b = tf.Variable(0., name = 'bias')

	optimizer = tf.train.GradientDescentOptimizer(learning_rate)
	cost_op = tf.reduce_sum(tf.square(Y - tf.multiply(X, w) - b))
	train_op = optimizer.minimize(cost_op)

	with tf.Session('grpc://localhost:' + args.server_port) as sess:
		sess.run(tf.global_variables_initializer())
		for epoch in range(max_epochs):
			for (x, y) in zip(train_X, train_Y):
				sess.run(train_op, feed_dict = {X: x, Y: y})
			if epoch % checkpoint_period == 0:
				print 'Epoch:', '%04d' % (epoch + 1), 'cost=', '{:.9f}'.format(sess.run(cost_op, feed_dict={X: train_X, Y: train_Y})), 'w=', sess.run(w), 'b=', sess.run(b)
		print 'Optimization Finished!'
		print 'cost=', '{:.9f}'.format(sess.run(cost_op, feed_dict={X: train_X, Y: train_Y})), 'w=', sess.run(w), 'b=', sess.run(b)

if __name__ == '__main__':
	main(parse_arguments(sys.argv[1:]))
```
通过不同的参数就可以去调用不同的服务器来完成相同的功能，如调用第一台worker：
```console
$ python client.py 2222
```
可以得到输出结果如下：
```console
Epoch: 0001 cost= 301.286193848 w= -0.853658 b= 9.79629
Epoch: 0051 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0101 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0151 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0201 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0251 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0301 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0351 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0401 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0451 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0501 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0551 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0601 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0651 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0701 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0751 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0801 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0851 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0901 cost= 10.734727859 w= 2.04805 b= 10.0511
Epoch: 0951 cost= 10.734727859 w= 2.04805 b= 10.0511
Optimization Finished!
cost= 10.734727859 w= 2.04805 b= 10.0511
```
## 多设备实践
#### 简介
搭建集群就是为了整合现实中的多台TensorFlow设备，不仅仅是只在多线程中去完成。于是，本次，共选择两台实际的设备进行多设备实践。
#### 集群配置
本次集群中仅一台工作机（192.168.10.200）：
```python
cluster_spec = { 'worker': ['192.168.10.200:2222'] }
```
#### 服务端
服务端位于工作机192.168.10.200上，代码如下：
```python
import tensorflow as tf
cluster_spec = { 'worker': ['192.168.10.200:2222'] }
cluster = tf.train.ClusterSpec(cluster_spec)
tf.train.Server(cluster, job_name='worker', task_index=0).join()
```
启动服务：
```console
$ python server.py
```
#### 客户端
客户端位于设备192.168.10.170上，代码如下：
```python
import tensorflow as tf
h = tf.constant('hello')
with tf.Session('grpc://192.168.10.200:2222') as s:
	s.run(h)
```
启动客户端：
```console
$ python client.py
```
能得到正确的结果，说明多设备实践成功。
## 总结
本次，对TensorFlow多机操作的简单探索取得了一定的进展：可以实现在任意一台安装了TensorFlow的设备上编写代码提交op到位于指定端口的服务端设备上进行执行。在此，TensorFlow的集群仅仅只是对设备进行了一个整合，客户端可以轻松调用到集群中的任意设备，具体调用哪些设备仍由客户端进行指定，集群中并没有看到有资源调度相关的实现。这便是我当前得出的结论，如果不当或错误之处，敬请指正，我也会在今后的研究中继续深入去探索。
