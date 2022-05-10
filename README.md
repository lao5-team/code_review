# Code Review 漫谈

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
- example 和 demo 中的代码片段会在使用者中传播，所以，错误和劣化的代码也会像病毒一样传播。

**请用一贯的高标准、严要求完成所有代码。**

### 【话题】一次 review 的工作量

过多改动是阻碍有效、快速 review 的最大敌人。

我遇到的“大型 code review”无外乎有两个结果：

* 拖很久完成 review，但是 reviewer 因为受不了持久战做出妥协，没守住底线。
* 拖很久不了了之，没有 review 完而直接 merge。

无论是哪种结果，都不是我们希望看到的。

**最佳实践：**

- **一次 review 的改动在 500 行以内，最多不超过 1000 行。**
- **一次 review 包含尽可能少的 commit。**
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

发起 code review 前请三思你的改动是否 diffable（suitable for processing by a diff program in order to show differences）。

### 【话题】如何埋“坑”

“对那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 TODO 注释. ”

要注意的是，TODO 注释的格式是（一定要带上埋“坑”人的 id）：

```c++
// TODO(kimi): what to do in future.
```

每个 TODO 注释都是一个“坑”。
我们允许“坑”的存在，但不允许“暗坑”的存在。
也不要让 TODO 注释没人认领，变成 NEVERDO。

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

当前上下文中不会引起歧义的缩写，也是好的缩写：

```c++
double mean = 0.0;
double var = 0.0;
// compute mean
// compute var
double stddev = sqrt(var);
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

## 反例

### 【案例】设计协议时，尽量不要用 boolean 作为参数类型

产品在设计第一版功能时，可能会提这样的需求：
“如果小明是男人，则穿短裤，是女人则穿裙子”。
“如果按钮支持单击，则显示蓝色，否则显示灰色”。

很多时候，这都是很不起眼的逻辑，开发者可能会想当然的把参数设计成 isMale, isSupportClick。

现在看起来一切运行得良好，但是如果程序发布到用户之后，产品发现满足不了用户的需求，于是修改了需求：
“如果小明是男人，则穿短裤，是女人则穿裙子，如果未填写性别，则穿长裤”。
“如果按钮支持单击，则显示蓝色，如果支持双击，则显示白色，两个都不支持，才显示灰色”。

这时候，可能就不得不把参数改成 int 类型，而往往不是所有的用户都会配合升级，那么和旧协议的兼容性问题就会让开发和测试抓狂。

对于这个问题，使用 int 作为参数类型会比 boolean 更有扩展性，比如用 gender，clickType 来代替 isMale, isSupportClick。设计参数时，不要主观的认为某个事物只存在相互对立的两面。
