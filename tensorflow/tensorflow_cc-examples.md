# TensorFlow C++ 示例代码

## 环境

- 系统：Ubuntu
- 版本：16.04.3 LTS
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器
- 显卡：索泰GTX1060 6G

## 写在前面

使用 C++ 语言来编写 TensorFlow 程序与使用 Python 语言一样，需要安装 TensorFlow 环境。参照[将 Tensorflow 源码编译成 C++ 库文件](./compilling-tensorflow-source-code-into-C++-library-file.md)这篇教程的安装步骤可以完成 TensorFlow C++ 环境的部署。

但是 C++ 源码需要编译后才能执行，编译所使用的 makefile 如下：

```makefile
target = tfcc_test
cc = g++ -std=c++11
include = -I/usr/local/tensorflow/include
lib = -L/usr/local/tensorflow/lib -ltensorflow_framework -ltensorflow_cc
flag = -Wl,-rpath=/usr/local/tensorflow/lib
source = ./src/main.cc

$(target): $(source)
	$(cc) $(source) -o $(target) $(include) $(lib) $(flag)

clean:
	rm $(target)

run:
	./$(target)
```

## 示例代码

- 创建 Session 

  ```c++
  #include "tensorflow/cc/client/client_session.h"

  using namespace tensorflow;

  int main()
  {
      auto root = Scope::NewRootScope();
      auto p_session = new ClientSession(root);
      delete p_session;
      return 0;
  }
  ```

- 常量

  ```c++
  #include "tensorflow/cc/client/client_session.h"
  #include "tensorflow/cc/ops/standard_ops.h"

  using namespace tensorflow;
  using namespace tensorflow::ops;
  using namespace std;

  int main()
  {
      auto root = Scope::NewRootScope();
      auto w = Const(root, 2, {});
      auto p_session = new ClientSession(root);
      vector<Tensor> outputs;
      p_session->Run({w}, &outputs);
      LOG(INFO) << "w = " << outputs[0].scalar<int>();
      delete p_session;
      return 0;
  }
  ```

- 变量

  ```c++
  #include "tensorflow/cc/client/client_session.h"
  #include "tensorflow/cc/ops/standard_ops.h"

  using namespace tensorflow;
  using namespace tensorflow::ops;
  using namespace std;

  int main()
  {
      auto root = Scope::NewRootScope();
      auto x = Variable(root, {}, DataType::DT_INT32);
      auto assign_x = Assign(root, x, 3); // initializer for x
      auto y = Variable(root, {2, 3}, DataType::DT_FLOAT);
      auto assign_y = Assign(root, y, RandomNormal(root, {2, 3}, DataType::DT_FLOAT)); // initializer for y
      auto p_session = new ClientSession(root);
      p_session->Run({assign_x, assign_y}, nullptr); // initialize
      vector<Tensor> outputs;
      p_session->Run({x, y}, &outputs);
      LOG(INFO) << "x = " << outputs[0].scalar<int>();
      LOG(INFO) << "y = " << outputs[1].matrix<float>();
      delete p_session;
      return 0;
  }
  ```

- 矩阵运算

  ```c++
  #include "tensorflow/cc/client/client_session.h"
  #include "tensorflow/cc/ops/standard_ops.h"

  using namespace tensorflow;
  using namespace tensorflow::ops;
  using namespace std;

  int main()
  {
      auto root = Scope::NewRootScope();
      auto x = Variable(root, {5, 2}, DataType::DT_FLOAT);
      auto assign_x = Assign(root, x, RandomNormal(root, {5, 2}, DataType::DT_FLOAT));
      auto y = Variable(root, {2, 3}, DataType::DT_FLOAT);
      auto assign_y = Assign(root, y, RandomNormal(root, {2, 3}, DataType::DT_FLOAT));
      auto xy = MatMul(root, x, y);
      auto z = Const(root, 2.f, {5, 3});
      auto xyz = Add(root, xy, z);
      auto p_session = new ClientSession(root);
      p_session->Run({assign_x, assign_y}, nullptr);
      vector<Tensor> outputs;
      p_session->Run({x, y, z, xy, xyz}, &outputs);
      LOG(INFO) << "x = " << outputs[0].matrix<float>();
      LOG(INFO) << "y = " << outputs[1].matrix<float>();
      LOG(INFO) << "xy = " << outputs[3].matrix<float>();
      LOG(INFO) << "z = " << outputs[2].matrix<float>();
      LOG(INFO) << "xyz = " << outputs[4].matrix<float>();
      delete p_session;
      return 0;
  }
  ```

- Placeholder

  ```c++
  #include "tensorflow/cc/client/client_session.h"
  #include "tensorflow/cc/ops/standard_ops.h"

  using namespace tensorflow;
  using namespace tensorflow::ops;
  using namespace std;

  int main()
  {
      auto root = Scope::NewRootScope();
      auto x = Placeholder(root, DataType::DT_INT32);
      auto w = Const(root, 1, {1, 2});
      auto wx = MatMul(root, x, w);
      auto b = Const(root, 2, {2});
      auto wx_b = Add(root, wx, b);
      auto p_session = new ClientSession(root);
      vector<Tensor> outputs;
      p_session->Run({{x, {{1}, {1}, {1}}}}, {wx, wx_b}, &outputs);
      LOG(INFO) << "wx = " << outputs[0].matrix<int>();
      LOG(INFO) << "wx_b = " << outputs[1].matrix<int>();
      delete p_session;
      return 0;
  }
  ```

