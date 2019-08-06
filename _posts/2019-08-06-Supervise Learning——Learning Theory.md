---
layout: post
title:  Supervise Learning——Learning Theory
date:   2019-08-06 17:50:43 +0800
categories: 课程
tag: 课程笔记
---

* content
{:toc}

# Learning Theory

> 主要讲述机器学习领域的基本理论，包括期望误差（泛化误差）与经验误差、经验风险最小化（ERM）、PAC可学习性和VC维等知识。

## 学习模型

学习模型这块主要包括了**期望误差**和**经验误差**的重新定义。

存在一个分布$\mathcal{D}$，里面的数据形式为$\mathcal{X} \times \mathcal{Y}$，$\mathcal{Y} = \{0,1\}$。学习所使用的数据是从$\mathcal{D}$中独立同分布采样出的。则对于函数$h : \mathcal{X} \times \mathcal{Y}$：

- **期望误差（也叫泛化误差）：** 
$$L_{\mathcal{D}}(h) = \mathcal{D} (\{ (x,y) : h(x) \ne y \}) = \mathbb{P}_{(x,y) \thicksim \mathcal{D}}[h(x) \ne y]$$

- **经验误差：** 经验误差是在训练集上的误差，在数据集$S = \{ (x_1, y_1), ... , (x_m, y_m)\}$上，经验误差为：
$$L_S(h) = \sum_{i=1}^m \frac{1}{m} [h(x_i) \ne y_i]$$

## 验证集边界

**定理：**

选定一个函数$h$，对任意的$\delta \in (0,1)$。从$\mathcal{D}$上随机采样得到一个大小为m的验证集V。有至少$1-\delta$的概率满足：
$$L_{\mathcal{D}}(h) \leq L_V(h) + \sqrt{\frac{\ln \frac{1}{\delta}}{2m}}$$

这个定理说明了泛化误差可以由在验证集V上的经验误差来界定。所以在实际中我们通过训练集学习出一个算法h，可以根据计算其在验证集上的误差，就可以来界定泛化误差。

## 经验风险最小化（$ERM_{\mathcal{H}}(S)$）

**输入：** 有限假设集$\mathcal{H}$、训练数据集$S=(x_1, y_1), ... , (x_m, y_m)$。

**定义经验风险：** $L_S(h) = \frac{1}{m} |\{ i : h(x_i) \ne y_i \}|$

**输出：** 使得$L_S(h)$最小的那个$h$（$h \in \mathcal{H}$）。

## Expected VS confident边界

> 这部分翻译PPT

对于一个有限的采样，泛化误差$L_\mathcal{D}(\mathcal{A}(S_m))$有一个基于算法、函数类别和样本集大小m的分布。

传统的统计方法关注的是**分布的平均值**，但是这个值可能会被误导。比如低折交叉验证。

在统计学习理论中，更倾向于关注分布的尾部，找到一个具有高概率的边界。

这看起来像一个统计测试——尤其是1%的置信度意味着在相同大小的随机样本上，结论不正确的概率小于1%。

在前面的乳腺癌数据示例中可以观察到，与分布的“尾部”移动相比，平均泛化误差只发生了少量的移动。

这也是缩写PAC的来源:大致正确，“置信度”参数$\delta$是我们被训练集误导的概率。

## PAC模型

### 可实现性假设

我们假设存在一个函数$f^\*$，对所有的$x \in \mathcal{X}$，都有$f^\*(x)=y$。（也就是一个不会出错的分类器）

这样假设是为了让$\mathcal{D}$只作为$\mathcal{X}$的分布（而不是$\mathcal{X} \times \mathcal{Y}$），因为Y我们可以直接通过$f^\* (\mathcal{X})$得到。

则泛化误差可以表示为：
$$L_{\mathcal{D}, f^\*}(h) = \mathbb{P}_{x \thicksim \mathcal{D}}[h(x) \ne f^\*(x)]$$

这种情况下，我们想找到一个算法$h = \mathcal{A}(S)$使得$L_{\mathcal{D}, f^\*}(h)$很小。

### PAC（参数有$\epsilon$和$\delta$）

学习器不知道$\mathcal{D}$和$f^\*$。

学习器接收精确度参数$\epsilon$和置信度参数$\delta$。

训练数据集S包含$m(\epsilon, \delta)$个样本，也就是说m的值取决于$\epsilon$和$\delta$，然后学习器在S上进行学习。

学习器输出一个学习到的函数h，这个函数应该有至少$1 - \delta$的置信度能够满足$L_{\mathcal{D}, f^\*}(h) \le \epsilon$。

满足了上面这个要求，就说明这个学习器是Probably($1 - \delta$的置信度) Approximately(误差最大为$\epsilon$)Correct的，即PAC的。

### NLF（没有免费的午餐）定理

**定理：**

取$\delta \in (0,1), \epsilon < 1/2$。对于任意的学习器A和训练集大小m，都必存在一个$\mathcal{D}, f^\*$，使得在大小为m的训练集上有至少$\delta$的概率满足$L_{\mathcal{D}, f^\*}(A(S)) \ge \epsilon$。

又有$L_{\mathcal{D}, f^\*}(随便猜) = 1/2$，所以这个定理说明了在没有先验知识的情况下，任何学习算法都不会一直表现的比随便猜好。

### 有限假设集上的学习

这里我们设$\mathcal{H}$为有限假设集。使用经验误差最小化，则有如下定理：

固定$\delta, \epsilon \in (0,1)$，如果m满足：
$$m \ge \frac{\log(|\mathcal{H}/\delta|)}{\epsilon}$$

则对于任意的$\mathcal{D}, f^\*$，都有至少$1-\delta$的概率（对于大小为m的随机采样训练集S）满足：
$$L_{\mathcal{D}, f^\*}(ERM_{\mathcal{H}} (S)) \le \epsilon$$

这里实际上说的就是在假设集有限的情况下，通过设定合适的训练集数量，可以使学习算法是PAC可学习的，故引出PAC可学习的概念。

**PAC可学习**

对于一个假设空间$\mathcal{H}$，如果存在一个函数$m_{\mathcal{H}} : (0,1)^2 \to \mathbb{N}$和一个学习器满足：

- 对于任意$\epsilon, \delta \in (0,1)$。
- 对于任意在$\mathcal{X}$上的分布$\mathcal{D}$，和任意标签函数$f^\* : \mathcal{X} \to (0,1)$。

当在一个样本数为$m \ge m_{\mathcal{H}}(\epsilon, \delta)$的训练集S上进行学习器的学习，学习器能够返回一个函数h使得有至少$1-\delta$的概率满足$L_{\mathcal{D}, f^\*}(h) \le \epsilon$。则称这个假设空间$\mathcal{H}$是PAC可学习的。

$m_{\mathcal{H}}$称为学习$\mathcal{H}$的**样本复杂度**。

### 样本复杂度

上一部分提到了$m_{\mathcal{H}}$为学习$\mathcal{H}$的**样本复杂度**。

如果$\mathcal{H}$是一个有限假设集，则由上一部分的第一个定理中的公式可得：若样本复杂度$m_{\mathcal{H}}(\epsilon, \delta) \le \frac{\log(|\mathcal{H}/\delta|)}{\epsilon}$，则$\mathcal{H}$是PAC可学习的。

> 样本复杂度可以由VC-维来表示。
ERM学习规则是一个通用的最优学习器。

## VC-维（理解定义、能够根据一个假设类计算VC-维）

先解释一个概念：**打散（Shatter）：**

定义$C=\{x_1, ... , x_{|C|}\} \subset \mathcal{X}$。

定义$\mathcal{H}_C = \{h_C:h \in \mathcal{H}\}$，其中$h_C:C \to \{-1,1\}$，且对于每个$x_i \in C$有$h_C(x_i)=h(x_i)$。（可以把$\mathcal{H}_C$理解成$\mathcal{H}$关于C的子集）

可以发现对于每个$h_C$都对应一个结果向量$(h(x_1), ... , h(x_{|C|})) \in \{\pm 1\}^{|C|}$，因此$|\mathcal{H}_C| \le 2^{|C|}$。

如果刚好满足$|\mathcal{H}_C| = 2^{|C|}$，我们就称$\mathcal{H}$**打散**了C。

定义了打散的概念之后，我们定义**VC-维**的概念：

$$VCdim(\mathcal{H}) = sup\{|C| : \mathcal{H} 打散 C\}$$

即VC-维是$\mathcal{H}$能够打散的集合C的最大size。

那么如果我们要证明$\mathcal{H}$的VC-维为d，我们需要证明两点：

1. 存在一个大小为d的集合C，它可以被$\mathcal{H}$打散。
2. 任意的大小为d+1的集合C都不能被$\mathcal{H}$打散。

**根据假设类计算VC-维，** 举个最简单的例子：

阈值函数：$\mathcal{X} = \mathbb{R}, \mathcal{H}=\{x \mapsto sign(x - \theta):\theta \in \mathbb{R}\}$。

1. 很明显$C = {0}$可以被打散。
2. 对于任意的两个x的C，都不能被打散，因为若$x_1 > x_2$，则不可能存在一个$h \in \mathcal{H}$使得$h(x_1) < 0而h(x_2) > 0$。

## VC-维（Large Margin Halfspaces，不知道怎么翻译，，，）

定义一个有界平方可求和序列：$l_2 = \{x \in \mathbb{R}^\infty : \sum_{i=0}^\infty x_i^2 < \infty\}$和一个内积运算$<x, x'> = \sum_{i=1}^\infty x_i x_i'$

