# TensorFlow 图像处理函数

### 图像编码处理

一张 RGB 色彩模式的图像可以看作一个三维矩阵，矩阵中的每一个数表示图像上不同位置，不同颜色的亮度。然而图像在存储时并不是直接记录这些矩阵中的数字，而是记录经过压缩编码之后的结果。所以要将一张图像还原成一个三维矩阵，需要解码的过程。 TensorFlow 提供了对 jpeg 和 png 格式图像的编码/解码函数。以下代码示范了如何使用 TensorFlow 对 jpeg 格式图像进行编码/解码。

```python
import tensorflow as tf

image_raw_data = tf.gfile.FastGFile('/path/to/picture', 'r').read()
with tf.Session() as sess:
    img_data = tf.image.decode_jpeg(image_raw_data)
    encoded_image = tf.image.encode_jpeg(img_data)
    with tf.gfile.GFile('path/to/output', 'wb') as f:
        f.write(encoded_image.eval())
```

### 图像大小调整

一般来说，现实中的图像大小是不固定的，但是神经网络的个数是固定的。所以在将网络的像素作为输入提供给神经网络之前，需要先将图像的大小统一。这就是图像大小调整需要完成的任务。

图像大小调整有两种方式，第一种是通过算法使得新的图像尽量保存原始图像上的所有信息。TensorFlow 提供了 4 种不同的方法，并且将它们封装到了 `tf.image.resize_images` 函数。以下代码示范了如何使用这个函数。

```python
# 处理前，将调整调整为实数类型，避免精度丢失。
img_data = tf.image.convert_image_dtype(img_data, dtype=tf.float32)
# 通过 tf.image.resize_images 函数调整图像的大小。这个函数第一个参数为原始图像，第二个参数为调整后图像的大小， method 参数给出了调整图像的算法。
# method=0: 双线性插值法（Bilinear interpolation）
# method=1: 最近邻居法（Nearest neighbor interpolation）
# method=2: 双三次插值法（Bicubic interpolation）
# method=3: 面积插值法（Area interpolation）
# 不同算法调整出来的结果会有细微差别，但不会相差太远。
resized = tf.image.resize_images(img_data, [300, 300], method=0)
```

图像大小调整的第二种方式，是通过算法对图像进行适当的裁剪或者填充。以下代码展示了通过 `tf.image.resize_image_with_crop_or_pad` 函数来调整图像大小的功能。这个函数的第一个参数为原始图像，第二个和第三个参数是调整后目标图像的大小。如果原始图像的尺寸大于目标图像，那么这个函数会自动截取原始图像中居中的部分。如果目标图像的尺寸大于原始图像，这个函数会自动在原始图像四周填充全 0 背景。

```python
# 自动截取（原始图像大小为 2000×2000）
croped = tf.image.resize_image_with_crop_or_pad(img_data, 1000, 1000)
# 自动填充（原始图像大小为 2000×2000）
padded = tf.image.resize_image_with_crop_or_pad(img_data, 3000, 3000)
```

TensorFlow 还支持通过比例调整图像大小，以下代码给出了一个样例。

```python
# 通过 tf.image.central_crop 函数按比例截取图像，第一个参数是原始图像，第二个参数为调整比例，这个比例需要一个 (0,1] 的实数。
central_croped = tf.image.central_crop(img_data, 0.5)
```

上面介绍的图像裁剪函数都是截取或者填充图像中间的部分。 TensorFlow 也提供了 `tf.image.crop_tobounding_box` 和 `tf.image.pad_to_bounding_box` 函数来裁剪或者填充给定区域的图像。这两个函数都要求给出的尺寸满足一定的眼球，否则程序会报错。比如在使用 `tf.image.crop_tobounding_box` 函数时， TensorFlow 要求提供的图像尺寸要大于目标尺寸，也就是要求原始图像能够裁剪出目标图像的大小。这里不再给出每个函数的具体样例，更多详情可以查询 TensorFlow 的 API 文档。

### 图像翻转

TensorFlow 提供了一些函数来支持对图像的翻转。以下代码实现了将图像上下翻转、左右翻转以及沿对角线翻转的功能。

```python
# 上下翻转
flipped = tf.image.flip_up_down(img_data)
# 左右翻转
flipped = tf.image.flip_left_right(img_data)
# 对角线翻转
flipped = tf.image.transpose_image(img_data)
```

在很多图像识别问题中，图像的翻转不会影响识别的结果。于是在训练图像识别的神经网络模型时，可以随机地翻转训练图像，这样训练得到的模型可以识别不同角度的实体。虽然这个问题可以通过收集更多的训练数据来解决，但是通过随机翻转训练图像的方式可以在零成本的情况下很大程度地缓解该问题。所以随机翻转训练图像是一种很常见的图像预处理方式。 TensorFlow 提供了方便的 API 完成随机图像翻转的过程。

```python
# 以 50% 的概率上下翻转图像
flipped = tf.image.random_flip_up_down(img_data)
# 以 50% 的概率左右翻转图像
flipped = tf.image.random_flip_left_right(img_data)
```
