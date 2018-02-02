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



## 编译



## 运行



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

