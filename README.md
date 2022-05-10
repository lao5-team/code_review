# Code Review

本文档遵守：

- [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)
- [文案风格指南](https://open.leancloud.cn/copywriting-style-guide/)

本文档记录 code review 过程中值得记录的点点滴滴和思考。

本文档和 [谷歌代码审核指南](https://jimmysong.io/eng-practices/docs/review) 穿插阅读效果更佳。

## 通用

### 【话题】部分代码完成度低、要求低

```
“这是测试，这么写省事儿，没必要关注安全问题吧”。
“这是测试，这段代码不需要写注释了吧”。
“这是例子，不需要考虑性能和安全性了吧”。
```

我经常遇到一种情况是：降低对 example、demo、test、benchmark 的要求。

这会造成以下不良影响：

- 代码会在开发者中传播、传承，糟糕的代码会增加各个环节的成本。
- example 和 demo 中的代码片段会在使用者中传播，错误的示范如同病毒。

**请用一贯的高标准、严要求完成所有代码。**

### 【话题】一次 review 的量

显而易见，过多改动是阻碍有效、快速 review 的最大敌人。

我遇到的“大型 code  review”无外乎有两个结果：

* 拖很久完成 review，但是 reviewer 因为受不了持久战做了妥协，没能守住底线。
* 拖很久不了了之直接 merge。
无论是哪种结果，都不是我们希望的。

**最佳实践：**

- **一次 review，改动行数在 500 行以内，最多不超过 1000 行。**
- **一次 review 尽量只包含一个 commit。**
    - **如果功能/需求复杂，可以按照合理的粒度拆分成多个 commit，但不要包含太多改动，不包含不相关的改动。**

### 【话题】有意义的 diff

无意义的 diff 是阻碍有效、快速 review 的另一大敌人。

什么是无意义的 diff 呢？

**场景一**

某项目代码格式不统一，某天决定统一格式，每个人配置代码格式化工具。
某同事修改了几十行代码，保存时代码格式化工具对代码格式化，最终产生上百行 diff。

**场景二**

某同事将 proto 文件和对应的 .pb.cc .pb.h 文件提交到 git。添加一个 proto 字段，最终产生几十行 diff。

**场景三**

某同事将二进制测试数据提交到 git。对它们的修改发起 code review。

前两个场景中，有意义的 diff 隐藏于大量无意义 diff。第三个场景中，二进制文件不方便 diff。

**最佳实践：**

- **保持项目统一的格式风格。**
    - **如果项目修改格式风格，统一修改并直接提交。**
- **不提交生成的代码（例如 protobuf、thrift、bison、flex 生成的代码）。**
- **尽可能不提交二进制文件。**
    - **频繁更新的二进制文件，请使用 git lfs。**
    - **考虑拆分二进制文件到另一个仓库，甚至不是代码库。**

Diffable（suitable for processing by a diff program in order to show differences）是有意义 diff 的充分必要条件。
发起 code review 前请三思你的改动是否 diffable。

### 【话题】优雅的埋“坑”

“对那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 TODO 注释. ”

要注意的是，TODO 注释的格式是（一定要带上埋“坑”人的 id）：

```c++
// TODO(kimi): what to do in future.
```

每个 TODO 注释都是一个“坑”。
我们允许“坑”的存在，但决不允许“暗坑”的存在，也不要让 TODO 没人认领，变成 NEVERDO。

## 可读性

好的可读性是优秀代码的第一要素，是 code review 应重点关注的。

### 命名

#### 【话题】缩写

**不使用奇怪的缩写。**

**不使用不约定俗成的缩写。**

**尽量不缩写短单词。**

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

#### 【案例】函数命名 1

```c++
// v1
void SetMetricsEnable(bool enable);
// v2
void SetEnableMetrics(bool enable);
```

Set 和 Enable 都是动词，“动名动”、“动动名”的词性组合都不适合做函数名。

```c++
// v1 改进
void SetMetricsEnabled(bool enabled);
```

v1 改进，可以勉强接受，但是不好。
SetMetricsEnabled 更像是 setter 函数，但实际上这是一个功能型函数。

```c++
// v3
void EnableMetrics(bool enable);
```

v3，enbale 为 false 的时候，到底是开启还是不开启呢？

```c++
// v4
void EnableMetrics();
void DisableMetrics();
// v5
void ToggleMetrics(bool enable);
```

**v4 和 v5 都是好的命名。**

## 一致性

好的一致性是优秀代码的第二要素，是 code review 应重点关注的。

## C++

## Java

## 其它
