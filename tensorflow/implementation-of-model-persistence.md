# 模型持久化实现

TensorFlow 提供了一个非常简单的 API 来保存和还原一个神经网络模型。这个 API 就是 `tf.train.Saver` 类。以下代码给出了保存 TensorFlow 计算图的方法。

```python
import tensorflow as tf

# 声明两个变量并计算它们的和。
v1 = tf.Variable(tf.constant(1.0, shape=[1]), name='v1')
v2 = tf.Variable(tf.constant(2.0, shape=[1]), name='v2')
result = v1 + v2

init_op = tf.global_variables_initializer()

# 声明 tf.train.Saver 类用于保存模型
saver = tf.train.Saver()

with tf.Session() as sess:
    sess.run(init_op)
    # 将模型保存到 /path/to/model/model.ckpt 文件。
    saver.save(sess, '/path/to/model/model.ckpt')
```

以上代码实现了持久化的一个简单的 TensorFlow 模型的功能。在这段代码中，通过 `saver.save` 函数将 TensorFlow 模型保存到了 /path/to/model/model.ckpt 文件中。 TensorFlow 模型一般会存在后缀为 .ckpt 的文件中。虽然以上程序只指定了一个文件路径，但是在这个文件目录下会出现三个文件。这是因为 TensorFlow 会将计算图的结构和图上参数取值分开保存。

上面这段代码会生成的第一个文件为 model.ckpt.meta ，它保存了 TensorFlow 计算图的结构，这里可以简单理解为神经网络的网络结构。第二个文件为 model.ckpt ，这个文件保存了 TensorFlow 程序中每一个变量的取值。最后一个文件为 checkpoint 文件，这个文件保存了一个目录下所有的模型文件列表。以下代码中给出了加载这个已经保存的 TensorFlow 模型的方法。

```python
import tensorflow as tf

# 使用和保存模型代码中一样的方式来声明变量。
v1 = tf.Variable(tf.constant(1.0, shape=[1]), name='v1')
v2 = tf.Variable(tf.constant(2.0, shape=[1]), name='v2')
result = v1 + v2

saver = tf.train.Saver()

with tf.Session() as sess:
    # 加载已经保存的模型，并通过已经保存的模型中变量的值来计算加法。
    saver.restore(sess, '/path/to/model/model.ckpt')
    print(sess.run(result))
```

这段加载模型的代码基本上和保存模型的代码是一样的。在加载模型的程序中也是先定义了 TensorFlow 计算图上的所有运算，并声明了一个 `tf.train.Saver` 类。两段代码唯一不同的是，在加载模型的代码中没有运行变量的初始化过程，而是将变量的值通过已经保存的模型加载进来。如果不希望重复定义图上的运算，也可以直接加载已经持久化的图。以下代码给出了一个样例。

```python
import tensorflow as tf

# 直接加载持久化的图。
saver = tf.train.import_meta_graph('/path/to/model/model.ckpt.meta')

with tf.Session() as sess:
    saver.restore(sess, '/path/to/model/model.ckpt')
    # 通过张量的名称来获取张量。
    print(sess.run(tf.get_default_graph().get_tensor_by_name('add:0'))) # 输出[3.]
```

在上面给出的程序中，默认保存和加载了 TensorFlow 计算图上定义好的全部变量。但有时可能只需要保存或者加载部分变量。比如，可能有一个之前训练好的五层神经网络模型，但现在想尝试一个六层的神经网络，那么可以将前面过五层神经网络中的参数直接加载到新的模型，而仅仅将最后一层神经网络重新训练。

为了保存或者加载部分变量，在声明 `tf.train.Saver` 类时可以提供一个列表来指定需要保存或者加载的变量。比如在加载模型的代码中使用 `saver=tf.train.Saver([v1])` 来构建 `tf.train.Saver` 类，那么只有变量 v1 会被加载进来。如果运行修改后只加载了 v1 的代码会得到变量未初始化的错误： `tensorflow.python.framework.errors.FailedPreconditionError: Attempting to use uninitialized value v2`

因为 v2 没有被加载，所以 v2 在运行初始化之前是没有值的。除了可以选取需要被加载的变量， `tf.train.Saver` 类也支持在保存或者加载时给变量重命名。下面给出了一个简单的样例程序说明变量重命名是如何被使用的。

```python
# 这里声明的变量名称和已经保存的模型中变量的名称不同。
v1 = tf.Variable(tf.constant(1.0, shape=[1]), name='other-v1')
v2 = tf.Variable(tf.constant(2.0, shape=[1]), name='other-v2')

# 如果直接使用 tf.train.Saver 来加载模型会报变量找不到的错误。错误信息如下：
# tensorflow.python.framework.errors.NotFoundError: Tensor name 'other-v1' not found in checkpoint files /path/to/model/model.ckpt

# 使用一个字典来重命名变量就可以加载原来的模型了。这个字典指定了原来名称为 v1 的变量现在加载到变量 v1 中（名称为 other-v1），名称为 v2 的变量加载到变量 v2 中（名称为 other-v2）。
saver = tf.train.Saver({'v1': v1, 'v2': v2})
```

在这个程序中，对变量 v1 和 v2 的名称进行了修改。如果直接通过 `tf.train.Saver` 默认的构造函数来加载保存的模型，那么程序会报变量找不到的错误。因为保存时候变量名称和加载时变量的名称不一致。为了解决这个问题， TensorFlow 可以通过字典将模型保存时的变量名和需要加载的变量联系起来。

这样做的目的之一是方便使用变量的滑动平均值（滑动平均值可以让神经网络模型更加健壮）。在 TensorFlow 中，每一个变量的滑动平均值是通过影子变量维护的，所以要获取变量的活动平均值实际上就是获取这个影子变量的取值。如果在加载模型时直接将影子变量映射到变量自身，那么在使用训练好的模型时就不需要再调用函数来获取变量的滑动平均值了。这样大大方便了滑动平均模型的使用。以下代码给出了一个保存滑动平均模型的样例。

```python
import tensorflow as tf

v = tf.Variable(0, dtype=tf.float32, name='v')
# 在没有申明滑动平均模型时只有一个变量 v ，所以以下语句只会输出 'v:0'。
for variable in tf.global_variables():
    print(variable.name)

ema = tf.train.ExponentialMovingAverage(0.99)
ema_avg_op = ema.apply(tf.global_variables())
# 在声明了滑动平均模型之后， TensorFlow 会自动生成一个影子变量 v/ExponentialMovingAverage 。于是以下语句会输出 'v:0' 和 'v/ExponentialMovingAverage:0' 。
for variable in tf.global_variables():
    print(variable.name)

init_op = tf.global_variables_initializer()

saver = tf.train.Saver()
with tf.Session() as sess:
    sess.run(init_op)

    sess.run(tf.assign(v, 10))
    sess.run(ema_avg_op)
    # 保存时， TensorFlow 会将 v:0 和 v/ExponentialMovingAverage:0 两个变量都保存下来。
    saver.save(sess, '/path/to/model/model.ckpt')
    print(sess.run([v, ema.average(v)])) # 输出 [10.0, 0.099999905]
```

以下代码给出了如何通过变量重命名直接读取变量的滑动平均值。从下面程序的输出可以看出，读取变量 v 的值实际上是上面代码中变量 v 的滑动平均值。通过这个方法，就可以使用完全一样的代码来计算滑动平均模型前向传播的结果。

```python
v = tf.Variable(0, dtype=tf.float32, name='v')
# 通过变量重命名将原来变量 v 的滑动平均值直接赋值给 v。
saver = tf.train.Saver({'v/ExponentialMovingAverage': v})
with tf.Session() as sess:
    saver.restore(sess, '/path/to/mdoel/model.ckpt')
    print(sess.run(v)) # 输出 0.099999905
```

为了方便加载时重命名滑动平均变量， `tf.train.ExponentialMovingAverage` 类提供了 `variables_to_restore` 函数来生成 `tf.train.Saver` 类所需要的变量重命名字典。以下代码给出了 `variables_to_restore` 函数的使用样例。

```python
import tensorflow as tf

v = tf.Variable(0, dtype=tf.float32, name='v')
ema = tf.train.ExponentialMovingAverage(0.99)

# 通过 variables_to_restore 函数可以直接生成上面代码中提供的字典 {'v/ExponentialMovingAverage': v}。
# 以下代码会输出：{'v/ExponentialMovingAverage': <tensorflow.Variable 'v:0' shape=() dtype=float32_ref>}。
# 其中后面的 Variable 类就代表了变量 v。
print(ema.variables_to_restore())

saver = tf.train.Saver(ema.variables_to_restore())
with tf.Session() as sess:
    saver.restore(sess, '/path/to/model/model.ckpt')
    print(sess.run(v)) # 输出 0.099999905
```

使用 `tf.train.Saver` 会保存运行 TensorFlow 程序所需要的全部信息，然而有时并不需要某些信息。比如在测试或者离线预测时，只需要知道如何从神经网络的输入层经过前向传播计算得到输出层即可，而不需要类似于变量初始化、模型保存等辅助节点的信息。而且，将变量取值和计算图结构分成不同的文件存储有时候也不方便，于是 TensorFlow 提供了 `convert_variables_to_constants` 函数，通过这个函数可以将计算图中的变量及其取值通过常量的方式保存，这样整个 TensorFlow 计算图可以统一存放在一个文件中。一下程序提供了一个样例。

```python
import tensorflow as tf
from tensorflow.python.framework import graph_util

v1 = tf.Variable(tf.constant(1.0, shape=[1]), name='v1')
v2 = tf.Variable(tf.constant(2.0, shape=[1]), name='v2')
result = v1 + v2

init_op = tf.global_variables_initializer()
with tf.Session() as sess:
    sess.run(init_op)
    # 导出当前计算图的 GraphDef 部分，只需要这一部分就可以完成从输入层到输出层的计算过程。
    graph_def = tf.get_dafault_graph().as_graph_def()

    # 将图中的变量及其取值转化为常量，同时将图中不必要的节点去掉，如果只关心程序中定义的某些计算时，和这些计算无关的节点就没有必要导出并保存了。在下面这行代码中，最后一个参数 ['add'] 给出了需要保存的节点名称。 add 节点是上面定义的两个变量相加的操作。注意这里给出的是计算节点的名称，所以没有后面的 :0 。
    out_graph_def = gtaph_util.convert_variables_to_constants(sess, graph_def, ['add'])
    # 将导出的模型存入文件。
    with tf.gfile.GFile('/path/to/model/combined_model.pb', 'wb') as f:
        f.write(out_graph_def.SerializeToString())
```

通过以下程序可以直接计算定义的加法运算的结果。当只需得到计算图中某个节点的取值时，这提供了一个更加方便的方法。

```python
import tensorflow as tf
from tensroflow.python.platform import gfile

with tf.Session() as sess:
    # 读取保存的模型文件，并将文件解析成对应的 GraphDef Protocol BUffer 。
    with gfile.FastGFile('/path/to/model/combined_model.pb', 'rb') as f:
        graph_def = tf.GraphDef()
        graph_def.ParseFromString(f.read())

    # 将 graph_def 中保存的图加载到当前图中。 return_elements=['add:0'] 给出了返回的张量的名称。在保存的时候给出的是计算节点的名称，所以为 'add' 。在加载的时候给出的是张量的名称，所以是 add:0 。
    result = tf.import_graph_def(graph_def, return_elements=['add:0'])
    print(sess.run(result)) # 输出 [3.0]
```
