# Code Review 漫谈

本文档遵守：

- [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)
- [文案风格指南](https://open.leancloud.cn/copywriting-style-guide/)

本文档记录 code review 过程中值得记录的点点滴滴和思考。

本文档和 [谷歌代码审核指南](https://jimmysong.io/eng-practices/docs/review) 穿插阅读效果更佳。

## 【通用】部分代码完成度低、要求低

> **场景一**
>
> “这是测试，这么写省事儿，没必要关注安全问题吧”。
>
> **场景二**
>
> “这是测试，这段代码不需要写注释了吧”。
>
> **场景三**
>
> “这是例子，不需要考虑性能和安全性了吧”。

我经常遇到一种情况是：降低对 example、demo、test、benchmark 的要求。

这会造成以下不良影响：

- 代码会在开发者中传播、传承，糟糕的代码会增加各个环节的成本。
- example 和 demo 中的代码片段会在使用者中传播，所以，错误和劣化的代码也会像病毒一样传播。

**最佳实践：**

- **请用一贯的高标准、严要求完成所有代码。**

## 【通用】一次 review 的工作量

过多改动是阻碍有效、快速 review 的最大敌人。

我遇到的“大型 code review”无外乎有两个结果：

* 拖很久完成 review，但是 reviewer 因为受不了持久战做出妥协，没守住底线。
* 拖很久不了了之，没有 review 完而直接 merge。

无论是哪种结果，都不是我们希望看到的。

**最佳实践：**

- **一次 review 的改动在 500 行以内，最多不超过 1000 行。**
- **一次 review 包含尽可能少的 commit。**
    - **如果功能/需求复杂，可以按照合理的粒度拆分成多个 commit，但不要包含太多改动，不包含不相关的改动。**

## 【通用】有意义的 diff

无意义的 diff 是阻碍有效、快速 review 的另一大敌人。

什么是无意义的 diff 呢？

> **场景一**
>
> 某项目代码格式不统一，某天决定统一格式，每个人配置代码格式化工具。
> 某同事修改了几十行代码，保存时代码格式化工具对代码格式化，最终产生上百行 diff。
>
>**场景二**
>
> 某同事将 proto 文件和对应的 .pb.cc .pb.h 文件提交到 git。添加一个 proto 字段，最终产生几十行 diff。
>
>**场景三**
>
> 某同事将二进制测试数据提交到 git。对它们的修改发起 code review。

前两个场景中，有意义的 diff 隐藏于大量无意义 diff。第三个场景中，二进制文件不方便 diff。

**最佳实践：**

- **保持项目统一的格式风格。**
    - **如果项目修改格式风格，统一修改并直接提交。**
- **不提交生成的代码（例如 protobuf、thrift、bison、flex 生成的代码）。**
- **尽可能不提交二进制文件。**
    - **频繁更新的二进制文件，请使用 git lfs。**
    - **考虑拆分二进制文件到另一个仓库，甚至不是代码库。**

发起 code review 前请三思你的改动是否 diffable（suitable for processing by a diff program in order to show differences）。

## 【通用】如何埋“坑”

“对那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 TODO 注释. ”

要注意的是，TODO 注释的格式是（一定要带上埋“坑”人的 id）：

```c++
// TODO(kimi): what to do in future.
```

每个 TODO 注释都是一个“坑”。
我们允许“坑”的存在，但不允许“暗坑”的存在。
也不要让 TODO 注释没人认领，变成 NEVERDO。

## 【可读性】缩写

**最佳实践：**

- **不使用奇怪的缩写。**
- **不使用不约定俗成的缩写。**
- **尽量不缩写短单词。**

坏的缩写：

```
thread -> thd
client -> clt
label -> lab
```

不太好的缩写：

```
count -> cnt
manager -> mgr
```

好的缩写：

```
advertisement -> ad
initialize -> init
implement -> impl
personalization -> p13n
```

当前上下文中不会引起歧义的缩写，也是好的缩写：

```c++
double mean = 0.0;
double var = 0.0;
// compute mean & var
double stddev = sqrt(var);
```

这里的 ```var``` 不会有人认为是 ```variable``` 的缩写。

## 【可读性】日志案例 1

关于日志，我遇到的一个典型问题是：失败后的错误日志揣测失败原因。

```c++
// v1
const char* file = ...;
int fd = open(file, ...);
if (fd == -1) {
  CARBON_ERROR("%s does not exist.", file);
}
```

v1 打开文件失败时，打印错误日志“文件不存在”。

事实上，导致打开文件失败的原因有很多，举几种情况：

1. 文件不存在。
2. 打开文件，但是打开了目录。
3. 没有读权限（读模式）。
4. 没有写权限（写模式）。
5. 打开了过多文件，即打开文件数超过了 ```ulimit -n```。

```c++
// v2
const char* file = ...;
int fd = open(file, ...);
if (fd == -1) {
  int e = errno;
  CARBON_ERROR("Failed to open(\"%s\"), errno=%d(%s).", file, e, strerror(e));
}
```

v2 是正确的做法。

**最佳实践：**

- **失败后的错误日志日志应该打印失败事件本身，不揣测失败原因。**
- **如果可以获取失败原因，请一起打印出来。**

## 【可读性】函数设计案例 1

某基础库要添加一个“开启/关闭 metrics 采集”的功能，考虑函数设计。

```c++
// v1
void SetMetricsEnable(bool enable);
// v2
void SetEnableMetrics(bool enable);
```

```Set``` 和 ```Enable``` 都是动词，“动名动”、“动动名”的词性不适合做函数名。

```enable``` 是动词，不适合做函数参数名。

```c++
// v1 改进
void SetMetricsEnabled(bool enabled);
```

v1 改进，可以接受，但是不好。
```SetMetricsEnabled``` 更像一个 setter 函数，但实际上这是一个功能型函数。

```c++
// v3
void EnableMetrics(bool enabled);
```

v3，enbaled 为 false 时，是开启还是关闭？

```c++
// v4
void EnableMetrics();
void DisableMetrics();
```

```c++
// v5
void ToggleMetrics(bool enabled);
```

v4 和 v5 都是好的函数声明。

## 【扩展性 & 可读性】函数设计案例 2

```Foo``` 处理 ```in```，返回处理后的结果。
如果 ```compressed``` 为 ```true```，```in``` 是用 Snappy 压缩的。

```c++
std::string Foo(const std::string& in, bool compressed = false);
```

有一天我们发现 LZ4 比 Snappy 压缩比更高、性能更好，然而 ```Foo``` 已经无法扩展了。

```Bar``` 比 ```Foo``` 有更好的扩展性：不但支持好了现状，而且可以扩展到未来。

```c++
enum COMPRESSION_TYPE {
  COMPRESSION_TYPE_NONE = 0,
  COMPRESSION_TYPE_SNAPPY = 1,
  COMPRESSION_TYPE_LZ4 = 2,
};
std::string Bar(const std::string& in,
                int compression_type = COMPRESSION_TYPE_NONE);
```

我们再看看它们的调用代码，可读性一目了然：

```c++
out = Foo(in, false);
out = Foo(in, true);
out = Bar(in, COMPRESSION_TYPE_NONE);
out = Bar(in, COMPRESSION_TYPE_SNAPPY);
out = Bar(in, COMPRESSION_TYPE_LZ4);
```

**最佳实践：**

- **设计函数时，考虑用 ```int/enum``` 取代 ```bool``` 做参数，它们有更好的扩展性和可读性。**

## 【扩展性 & 可读性】函数设计案例 3

```Init``` 根据若干参数初始化某模块。

```c++
// v1
bool Init(const std::string& model_dir,
          int session_pool_max_size = 96,
          int intra_op_parallelism_threads = 1,
          int inter_op_parallelism_threads = -1,
          bool enable_warmup = true,
          bool enable_session_run_timeout = true);
```

该模块迭代到第二版，引入 CUDA 和 OpenCL 能力，添加对应参数 ```enable_cuda``` 和 ```enable_opencl```。

如此迭代下去，该模块还将引入多少参数呢？

```c++
// v2
bool Init(const std::string& model_dir,
          int session_pool_max_size = 96,
          int intra_op_parallelism_threads = 1,
          int inter_op_parallelism_threads = -1,
          bool enable_warmup = true,
          bool enable_session_run_timeout = true,
          bool enable_cuda = false,
          bool enable_opencl = false);
```

第二版可以这样改进：通过 string map 传递所有参数，彻底解决参数扩展性问题。

```c++
// v2 改进
template <typename T>
bool GetOption(const std::unordered_map<std::string, std::string>& options,
               const std::string& name, T* option);
template <typename T>
bool GetOption(const std::unordered_map<std::string, std::string>& options,
               const std::string& name, T* option, const T& default_option);

bool Init(const std::unordered_map<std::string, std::string>& options) {
  std::string model_dir;
  int session_pool_max_size;
  int intra_op_parallelism_threads;
  int inter_op_parallelism_threads;
  bool enable_warmup;
  bool enable_session_run_timeout;
  bool enable_cuda;
  bool enable_opencl;
  if (!GetOption(options, "model_dir", &model_dir) ||
      !GetOption(options, "session_pool_max_size", &session_pool_max_size,
                 96) ||
      !GetOption(options, "intra_op_parallelism_threads",
                 &intra_op_parallelism_threads, 1) ||
      !GetOption(options, "inter_op_parallelism_threads",
                 &inter_op_parallelism_threads, -1) ||
      !GetOption(options, "enable_warmup", &enable_warmup, true) ||
      !GetOption(options, "enable_session_run_timeout",
                 &enable_session_run_timeout, true) ||
      !GetOption(options, "enable_cuda", &enable_cuda, false) ||
      !GetOption(options, "enable_opencl", &enable_opencl, false)) {
    return false;
  }

  // ...
  return true;
}
```

我们再看看 ```Init``` 的调用代码，可读性一目了然：

```c++
// v1
bool ok = Init("mandatory_model_dir", 96, 1, -1, true, true);
```

```c++
// v2
bool ok = Init("mandatory_model_dir", 96, 1, -1, true, true, false, true);
```

```c++
// v2 改进
std::unordered_map<std::string, std::string> options;
options["model_dir"] = "mandatory_model_dir";
options["session_pool_max_size"] = "96";
options["intra_op_parallelism_threads"] = "1";
options["inter_op_parallelism_threads"] = "-1";
options["enable_warmup"] = "true";
options["enable_session_run_timeout"] = "true";
options["enable_cuda"] = "false";
options["enable_opencl"] = "false";
bool ok = Init(options);
```

**最佳实践：**

- **设计函数时，尽量避免过多参数，考虑用特殊方式处理大量参数的传递。**
