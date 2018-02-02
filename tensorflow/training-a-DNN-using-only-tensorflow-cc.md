# 仅用TensorFlow C++训练一个DNN

## 环境

- 系统：Ubuntu
- 版本：16.04.3 LTS
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器
- 显卡：索泰GTX1060 6G

## 写在前面

使用C++语言来使用TensorFlow与使用Python语言一样，需要安装TensorFlow环境。参照[将Tensorflow源码编译成C++库文件](./compilling-tensorflow-source-code-into-C++-library-file.md)这篇教程的安装步骤可以完成TensorFlow C++环境的部署。

本文目标是使用C++调用TensorFlow建立一个线性回归模型，可能模型本身并不具有吸引力，但可以据此走进C++调用TensorFlow的大门。

## 编码

#### 引入头文件

1. 定义模型需要使用TensorFlow的基本操作，引入：`#include "tensorflow/cc/ops/standard_ops.h"`
2. 训练过程中有梯度的相关操作，引入：`#include "tensorflow/cc/framework/gradients.h"`
3. 开始训练需要创建Session，引入：`#include "tensorflow/cc/client/client_session.h"`

#### 数据

围绕方程$y=2x+0.5$创建100个点作为数据集：

```c++
// benchmark
auto benchmark_w = 2.0, benchmark_b = 0.5;

// data
auto nof_samples = 100;
struct Sample 
{
    float sample;
    float label;
};
std::vector<struct Sample> dataset;
std::srand((unsigned)std::time(NULL));
for (int i = 0; i < nof_samples; i++)
{
    float sample = std::rand() / float(RAND_MAX) - 0.5;
    float label = benchmark_w * sample + benchmark_b + std::rand() / float(RAND_MAX) * 0.01;
    dataset.push_back({sample, label});
}
```

#### 模型

TensorFlow C++的独特之处在于，需要一个 Scope 对象来保持构建静态计算图的状态，并将该对象传递给每个操作。

```c++
tensorflow::Scope root = tensorflow::Scope::NewRootScope();
```

建立占位符`x`和`y`用于代表每个样本的内容与标签。

```c++
auto x = tensorflow::ops::Placeholder(root, tensorflow::DataType::DT_FLOAT);
auto y = tensorflow::ops::Placeholder(root, tensorflow::DataType::DT_FLOAT);
```

建立权重`w`和偏置`b`。在C++中，定义一个变量之后，必须再定义一个Assign节点来为该变量分配默认值。对于权重，使用`RandomNormal`来分配一个服从正态分布的随机值，对于偏置，使用常数分配默认值。

```c++
auto w = tensorflow::ops::Variable(root, {1, 1}, tensorflow::DataType::DT_FLOAT);
auto assign_w = tensorflow::ops::Assign(root, w, tensorflow::ops::RandomNormal(root, {1, 1}, tensorflow::DataType::DT_FLOAT));
auto b = tensorflow::ops::Variable(root, {1, 1}, tensorflow::DataType::DT_FLOAT);
auto assign_b = tensorflow::ops::Assign(root, b, {{0.0f}});
```

使用TensorFlow的基本操作定义网络，得到预测值的计算方式。

```c++
auto y_ = tensorflow::ops::Add(root, tensorflow::ops::MatMul(root, w, x), b);
```

以预测值与实际值的差异作为损失函数。

```c++
auto loss = tensorflow::ops::L2Loss(root, tensorflow::ops::Sub(root, y_, y));
```

至此，完成了网络的前向传播，开始进行反向传播。但C++中还没有像Python中Optimizer一样的函数定义，所以需要手动完成这一过程。

首先，在计算图中加入梯度运算。

```c++
std::vector<tensorflow::Output> grad_outputs;
TF_CHECK_OK(AddSymbolicGradients(root, {loss}, {w, b}, &grad_outputs));
```

接着，依据一定的学习率将梯度变化应用与各个变量。

```c++
auto learn_rate = tensorflow::ops::Cast(root, 0.01, tensorflow::DataType::DT_FLOAT);
auto apply_w = tensorflow::ops::ApplyGradientDescent(root, w, learn_rate, {grad_outputs[0]});
auto apply_b = tensorflow::ops::ApplyGradientDescent(root, b, learn_rate, {grad_outputs[1]});
```

至此，模型就建立完成了。

#### 训练

创建Session。

```c++
tensorflow::ClientSession sess(root);
```

初始化变量。在Python中有`global_variables_initializer`这样的初始化操作，但是C++中，需要逐一对变量初始化。

```c++
sess.Run({assign_w, assign_b}, nullptr);
```

输入数据执行训练，建立outputs变量接收结果。

```
std::vector<tensorflow::Tensor> outputs;
TF_CHECK_OK(sess.Run({{x, {{dataset[i].sample}}}, {y, {{dataset[i].label}}}}, {w, b, loss, apply_w, apply_b}, &outputs));
```

## 编译

建立makefile来定义相关规则。

```makefile
target = tfcc_linear_regression
cc = g++ -std=c++11
include = -I/usr/local/tensorflow/include
lib = -L/usr/local/tensorflow/lib -ltensorflow_framework -ltensorflow_cc
flag = -Wl,-rpath=/usr/local/tensorflow/lib
source = ./main.cc

$(target): $(source)
	$(cc) $(source) -o $(target) $(include) $(lib) $(flag)

clean:
	rm $(target)

run:
	./$(target)
```

命令终端执行`make`完成编译。

## 运行

命令终端执行`make run`开始运行。运行日志如下：

```
2018-02-02 16:30:46.080189: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:898] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
2018-02-02 16:30:46.080398: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1208] Found device 0 with properties:
name: GeForce GTX 1060 6GB major: 6 minor: 1 memoryClockRate(GHz): 1.7845
pciBusID: 0000:01:00.0
totalMemory: 5.93GiB freeMemory: 5.42GiB
2018-02-02 16:30:46.080413: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1308] Adding visible gpu devices: 0
2018-02-02 16:30:46.226059: I tensorflow/core/common_runtime/gpu/gpu_device.cc:989] Creating TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 5200 MB memory) -> physical GPU (device: 0, name: GeForce GTX 1060 6GB, pci bus id: 0000:01:00.0, compute capability: 6.1)
2018-02-02 16:30:46.392500: I ./main.cc:55] epoch 1: w=0.0287388 b=0.308781 loss=0.00985585
2018-02-02 16:30:46.413638: I ./main.cc:55] epoch 2: w=0.174093 b=0.425818 loss=0.0747276
2018-02-02 16:30:46.435709: I ./main.cc:55] epoch 3: w=0.307654 b=0.482579 loss=0.152716
2018-02-02 16:30:46.457657: I ./main.cc:55] epoch 4: w=0.434336 b=0.479716 loss=0.0029346
2018-02-02 16:30:46.476430: I ./main.cc:55] epoch 5: w=0.548934 b=0.490159 loss=0.0811246
2018-02-02 16:30:46.501751: I ./main.cc:55] epoch 6: w=0.65713 b=0.496897 loss=0.00255468
2018-02-02 16:30:46.523515: I ./main.cc:55] epoch 7: w=0.756212 b=0.496299 loss=0.0120363
2018-02-02 16:30:46.546352: I ./main.cc:55] epoch 8: w=0.84801 b=0.504326 loss=0.0147831
2018-02-02 16:30:46.567823: I ./main.cc:55] epoch 9: w=0.933083 b=0.503594 loss=0.0125665
2018-02-02 16:30:46.590647: I ./main.cc:55] epoch 10: w=1.01141 b=0.497705 loss=0.0388892
2018-02-02 16:30:46.613802: I ./main.cc:55] epoch 11: w=1.08457 b=0.500842 loss=0.028814
2018-02-02 16:30:46.636694: I ./main.cc:55] epoch 12: w=1.15258 b=0.502042 loss=0.00993665
2018-02-02 16:30:46.659202: I ./main.cc:55] epoch 13: w=1.215 b=0.501114 loss=0.0153127
2018-02-02 16:30:46.682020: I ./main.cc:55] epoch 14: w=1.27175 b=0.494934 loss=0.0623955
2018-02-02 16:30:46.705282: I ./main.cc:55] epoch 15: w=1.32698 b=0.504595 loss=0.00330505
2018-02-02 16:30:46.726798: I ./main.cc:55] epoch 16: w=1.37642 b=0.502182 loss=0.0132433
2018-02-02 16:30:46.750086: I ./main.cc:55] epoch 17: w=1.42283 b=0.507818 loss=0.000372095
2018-02-02 16:30:46.773001: I ./main.cc:55] epoch 18: w=1.46551 b=0.506062 loss=0.000904494
2018-02-02 16:30:46.796304: I ./main.cc:55] epoch 19: w=1.50506 b=0.50463 loss=5.37055e-08
2018-02-02 16:30:46.823429: I ./main.cc:55] epoch 20: w=1.5416 b=0.503685 loss=0.001798
2018-02-02 16:30:46.842421: I ./main.cc:55] epoch 21: w=1.57475 b=0.50232 loss=0.0177823
2018-02-02 16:30:46.864284: I ./main.cc:55] epoch 22: w=1.60652 b=0.504101 loss=0.00856935
2018-02-02 16:30:46.883906: I ./main.cc:55] epoch 23: w=1.63581 b=0.502651 loss=0.00331169
2018-02-02 16:30:46.905470: I ./main.cc:55] epoch 24: w=1.6621 b=0.503855 loss=0.0134963
2018-02-02 16:30:46.925218: I ./main.cc:55] epoch 25: w=1.68738 b=0.503722 loss=0.00745212
2018-02-02 16:30:46.945532: I ./main.cc:55] epoch 26: w=1.71044 b=0.507803 loss=0.00680165
2018-02-02 16:30:46.963771: I ./main.cc:55] epoch 27: w=1.73228 b=0.505841 loss=0.000649392
2018-02-02 16:30:46.984000: I ./main.cc:55] epoch 28: w=1.75184 b=0.504092 loss=0.00363269
2018-02-02 16:30:47.008261: I ./main.cc:55] epoch 29: w=1.77049 b=0.505138 loss=3.02402e-05
2018-02-02 16:30:47.028533: I ./main.cc:55] epoch 30: w=1.78746 b=0.505603 loss=5.24845e-05
2018-02-02 16:30:47.051045: I ./main.cc:55] epoch 31: w=1.80317 b=0.505201 loss=0.000309116
2018-02-02 16:30:47.072219: I ./main.cc:55] epoch 32: w=1.81768 b=0.506146 loss=0.000825671
2018-02-02 16:30:47.092352: I ./main.cc:55] epoch 33: w=1.8312 b=0.505209 loss=0.000545735
2018-02-02 16:30:47.115795: I ./main.cc:55] epoch 34: w=1.84374 b=0.503566 loss=9.39509e-05
2018-02-02 16:30:47.136988: I ./main.cc:55] epoch 35: w=1.85532 b=0.503735 loss=0.000219835
2018-02-02 16:30:47.159452: I ./main.cc:55] epoch 36: w=1.86586 b=0.503876 loss=0.00144605
2018-02-02 16:30:47.180647: I ./main.cc:55] epoch 37: w=1.87578 b=0.503936 loss=0.00156384
2018-02-02 16:30:47.200503: I ./main.cc:55] epoch 38: w=1.88501 b=0.504044 loss=0.0012373
2018-02-02 16:30:47.222264: I ./main.cc:55] epoch 39: w=1.89369 b=0.504043 loss=0.000211598
2018-02-02 16:30:47.245331: I ./main.cc:55] epoch 40: w=1.90162 b=0.503942 loss=7.68242e-05
2018-02-02 16:30:47.267593: I ./main.cc:55] epoch 41: w=1.90888 b=0.5045 loss=0.000247896
2018-02-02 16:30:47.289457: I ./main.cc:55] epoch 42: w=1.91568 b=0.504379 loss=9.1548e-05
2018-02-02 16:30:47.313437: I ./main.cc:55] epoch 43: w=1.92192 b=0.50485 loss=5.86275e-05
2018-02-02 16:30:47.336081: I ./main.cc:55] epoch 44: w=1.92773 b=0.503919 loss=5.96701e-06
2018-02-02 16:30:47.358377: I ./main.cc:55] epoch 45: w=1.93311 b=0.503887 loss=1.33136e-05
2018-02-02 16:30:47.382712: I ./main.cc:55] epoch 46: w=1.93802 b=0.503764 loss=0.000184997
2018-02-02 16:30:47.403499: I ./main.cc:55] epoch 47: w=1.94269 b=0.504512 loss=2.31828e-07
2018-02-02 16:30:47.427308: I ./main.cc:55] epoch 48: w=1.94696 b=0.504091 loss=8.57235e-06
2018-02-02 16:30:47.448180: I ./main.cc:55] epoch 49: w=1.95079 b=0.504129 loss=0.000393243
2018-02-02 16:30:47.468167: I ./main.cc:55] epoch 50: w=1.95457 b=0.504473 loss=3.21638e-05
2018-02-02 16:30:47.489507: I ./main.cc:55] epoch 51: w=1.95789 b=0.504286 loss=0.000208695
2018-02-02 16:30:47.508765: I ./main.cc:55] epoch 52: w=1.96113 b=0.504365 loss=2.6422e-07
2018-02-02 16:30:47.528058: I ./main.cc:55] epoch 53: w=1.96397 b=0.504467 loss=0.00013571
2018-02-02 16:30:47.547171: I ./main.cc:55] epoch 54: w=1.96667 b=0.504267 loss=0.000163676
2018-02-02 16:30:47.569640: I ./main.cc:55] epoch 55: w=1.96922 b=0.504218 loss=1.17267e-05
2018-02-02 16:30:47.593821: I ./main.cc:55] epoch 56: w=1.97154 b=0.504363 loss=1.80367e-06
2018-02-02 16:30:47.613781: I ./main.cc:55] epoch 57: w=1.97366 b=0.504471 loss=5.36514e-05
2018-02-02 16:30:47.635224: I ./main.cc:55] epoch 58: w=1.97565 b=0.504339 loss=4.07987e-05
2018-02-02 16:30:47.656690: I ./main.cc:55] epoch 59: w=1.9775 b=0.504332 loss=2.36653e-05
2018-02-02 16:30:47.676707: I ./main.cc:55] epoch 60: w=1.97923 b=0.504351 loss=1.64737e-06
2018-02-02 16:30:47.699713: I ./main.cc:55] epoch 61: w=1.9808 b=0.504481 loss=4.04759e-06
2018-02-02 16:30:47.719521: I ./main.cc:55] epoch 62: w=1.98227 b=0.504226 loss=3.48166e-07
2018-02-02 16:30:47.742122: I ./main.cc:55] epoch 63: w=1.98362 b=0.504258 loss=2.83619e-06
2018-02-02 16:30:47.764712: I ./main.cc:55] epoch 64: w=1.98486 b=0.504377 loss=2.76433e-05
2018-02-02 16:30:47.764737: I ./main.cc:58] elapsed time： 1.49406s
```



## 全部源码

```c++
#include "tensorflow/cc/client/client_session.h"
#include "tensorflow/cc/ops/standard_ops.h"
#include "tensorflow/cc/framework/gradients.h"

int main()
{
    // benchmark
    auto benchmark_w = 2.0, benchmark_b = 0.5;

    // data
    auto nof_samples = 100;
    struct Sample 
    {
        float sample;
        float label;
    };
    std::vector<struct Sample> dataset;
    std::srand((unsigned)std::time(NULL));
    for (int i = 0; i < nof_samples; i++)
    {
        float sample = std::rand() / float(RAND_MAX) - 0.5;
        float label = benchmark_w * sample + benchmark_b + std::rand() / float(RAND_MAX) * 0.01;
        dataset.push_back({sample, label});
    }

    // model
    tensorflow::Scope root = tensorflow::Scope::NewRootScope();
    auto x = tensorflow::ops::Placeholder(root, tensorflow::DataType::DT_FLOAT);
    auto y = tensorflow::ops::Placeholder(root, tensorflow::DataType::DT_FLOAT);
    auto w = tensorflow::ops::Variable(root, {1, 1}, tensorflow::DataType::DT_FLOAT);
    auto assign_w = tensorflow::ops::Assign(root, w, tensorflow::ops::RandomNormal(root, {1, 1}, tensorflow::DataType::DT_FLOAT));
    auto b = tensorflow::ops::Variable(root, {1, 1}, tensorflow::DataType::DT_FLOAT);
    auto assign_b = tensorflow::ops::Assign(root, b, {{0.0f}});
    auto y_ = tensorflow::ops::Add(root, tensorflow::ops::MatMul(root, w, x), b);
    auto loss = tensorflow::ops::L2Loss(root, tensorflow::ops::Sub(root, y_, y));
    std::vector<tensorflow::Output> grad_outputs;
    TF_CHECK_OK(AddSymbolicGradients(root, {loss}, {w, b}, &grad_outputs));
    auto learn_rate = tensorflow::ops::Cast(root, 0.01, tensorflow::DataType::DT_FLOAT);
    auto apply_w = tensorflow::ops::ApplyGradientDescent(root, w, learn_rate, {grad_outputs[0]});
    auto apply_b = tensorflow::ops::ApplyGradientDescent(root, b, learn_rate, {grad_outputs[1]});

    // train
    tensorflow::ClientSession sess(root);
    sess.Run({assign_w, assign_b}, nullptr);
    std::vector<tensorflow::Tensor> outputs;
    timespec t0, t1;
    clock_gettime(CLOCK_MONOTONIC, &t0);
    for (int epoch = 1; epoch <= 64; epoch++)
    {
        std::random_shuffle(dataset.begin(), dataset.end());
        for (int i = 0; i < nof_samples; i++)
        {
            TF_CHECK_OK(sess.Run({{x, {{dataset[i].sample}}}, {y, {{dataset[i].label}}}}, {w, b, loss, apply_w, apply_b}, &outputs));
        }
        LOG(INFO) << "epoch " << epoch << ": w=" << outputs[0].matrix<float>() << " b=" << outputs[1].matrix<float>() << " loss=" << outputs[2].scalar<float>();
    }
    clock_gettime(CLOCK_MONOTONIC, &t1);
    LOG(INFO) << "elapsed time： " << t1.tv_sec - t0.tv_sec + (t1.tv_nsec - t0.tv_nsec) * 1.0 / 1000000000 << "s";
    return 0;
}
```

