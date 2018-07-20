# 多 GPU 数据并行

## 环境

- 系统：Ubuntu
- 版本：16.04.3 LTS
- 类型：64位操作系统 64位处理器
- 显卡：GTX1080 * 3
- TensorFlow 版本：1.8.0

## 写在前面

本教程将介绍在一台机器的多个 GPU 上并行训练深度学习模型。因为一般来说一台机器上的多个 GPU 性能相似，所以在这种设置下会更多地采用同步模型来进行训练，下面将结合代码对训练的整个过程进行讲解。

## 并行

引入 tensorflow 库文件。

```python
import tensorflow as tf
```

引入训练数据和定义好的模型。

```python
from data import input as c10_input
from inference.lenet5 import inference as c10_inference
```

定义训练超参数。

```python
flags = tf.app.flags
flags.DEFINE_integer('train_steps', 100000, 'number of training steps.')
flags.DEFINE_integer('batch_size', 64, 'number of examples per step.')
flags.DEFINE_float('lr_base', 0.001, 'base learning rate.')
flags.DEFINE_float('lr_decay', 0.99, 'learning rate decay.')
flags.DEFINE_float('reg_rate', 0.0001, 'regularization rate on model weights.')
flags.DEFINE_float('ma_decay', 0.99, 'moving average decay.')
flags.DEFINE_integer('nof_gpus', 3, 'how many GPUs to use.')
FLAGS = flags.FLAGS
```

定义主函数，开始定义计算图谱。

```python
def main(_):
    c10_input.extract()
    graph = tf.Graph()
```

使用 CPU 读取数据并执行预处理操作。

```python
    with graph.as_default(), tf.device('/cpu:0'):
        # Get images and labels for CIFAR-10.
        images, labels = c10_input.distorted_inputs(batch_size=FLAGS.batch_size, image_size=[28, 28])
        labels = tf.one_hot(labels, 10)
```

定义优化器用于执行梯度更新。

```python
        # Create an optimizer that performs gradient descent.
        global_step = tf.train.get_or_create_global_step()
        learning_rate = tf.train.exponential_decay(FLAGS.lr_base, global_step, 64, FLAGS.lr_decay)
        optimizer = tf.train.AdamOptimizer(learning_rate)
```

各个 GPU 并行计算梯度更新值。

```python
        # Collect gradients from every gpu.
        tower_grads = []
        accs = []
        losses = []
        reuse_variable = False
        for i in xrange(FLAGS.nof_gpus):
            with tf.device('/gpu:%d' % i):
                with tf.variable_scope(tf.get_variable_scope(), reuse=reuse_variable):
                    logits = c10_inference(images, tf.contrib.layers.l2_regularizer(FLAGS.reg_rate))
                # Accuracy_op
                pred = tf.equal(tf.argmax(logits, 1), tf.argmax(labels, 1))
                acc = tf.reduce_mean(tf.cast(pred, tf.float32))
                accs.append(acc)
                cross_entropy = tf.nn.sparse_softmax_cross_entropy_with_logits(logits=logits, labels=tf.argmax(labels, 1))
                loss = tf.reduce_mean(cross_entropy) + tf.add_n(tf.get_collection('losses'))
                grads = optimizer.compute_gradients(loss)
                losses.append(loss)
                tower_grads.append(grads)
                reuse_variable = True
```

计算平均准确率、平均 Loss 、平均梯度值。

```python
        acc_op = tf.reduce_mean(accs)
        loss_op = tf.reduce_mean(losses)
        
        # Compute average gradients.
        average_grads = []
        for grad_and_vars in zip(*tower_grads):
            grads = []
            for g, _ in grad_and_vars:
                grads.append(tf.expand_dims(g, 0))
            grad = tf.reduce_mean(tf.concat(grads, 0), 0)
            average_grads.append((grad, grad_and_vars[0][1]))
```

应用梯度值。

```python
        # Apply gradients.
        train_step = optimizer.apply_gradients(average_grads, global_step=global_step)
```

计算滑动平均模型。

```python
        # Compute moving average.
        var_avg = tf.trainable_variables() + tf.moving_average_variables()
        var_avg_op = tf.train.ExponentialMovingAverage(FLAGS.ma_decay, global_step).apply(var_avg)
```

定义 train_op 用于迭代。

```python
        # Update per train step.
        train_op = tf.group(train_step, var_avg_op)
```

定义初始化 op ，并结束图谱定义。

```python
        # Build an initialization operation to run below.
        init_op = tf.global_variables_initializer()
        graph.finalize()
```

配置 Session 。

```python
    # Config session.
    config = tf.ConfigProto()
    config.allow_soft_placement = True
    config.gpu_options.allow_growth = True
```

执行迭代。

```python
    # Training...
    with tf.Session(graph=graph, config=config) as sess:
        print('%s: Session started.' % (dt.datetime.now()))
        sess.run(init_op)
        coord = tf.train.Coordinator()
        threads = tf.train.start_queue_runners(coord=coord, sess=sess)
        while not coord.should_stop():
            _, loss, acc, steps = sess.run([train_op, loss_op, acc_op, global_step])
            if steps % 100 == 0:
                print('%s: After %d steps training, loss / accuracy on training batch is %g / %g.' % (dt.datetime.now(), steps, loss, acc))
            if steps == FLAGS.train_steps:
                coord.request_stop()
        coord.join(threads)
        print('%s: Finished training.' % (dt.datetime.now()))
```

调用主函数。

```python
if __name__ == '__main__':
    tf.app.run(main=main)
```

## 结语

通过调整超参数 `nof_gpus` ，可以实验同步模式下随着 GPU 个数的增加训练速度的加速比率。通过实验可以得到这样的结论：**同步模式下，当 GPU 数量增加时，虽然加速比率不是线性增长，但 TensorFlow 仍然可以通过增加 GPU 数量来有效地加速深度学习模型地训练过程。**
