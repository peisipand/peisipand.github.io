---
layout:     post
title:      最大似然估计、最大后验概率估计、最小二乘法和贝叶斯定理
subtitle:   MLE, MAP, LSE and Bayes' Theorem
date:       2024-07-06
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Mathematics
---

> 理工科的学习中，无论是机器学习还是大气反演，都可以视为是参数估计问题， 即如何找到 符合观测/符合训练集 的最优的参数。经常看到的算法有：最小二乘，最小二乘的各种改进版本，贝叶斯估计等。关于这些理论的讲解和博客网上非常多，但很难找到一个详细且易于理解的总结。通过近几年的不断学习，对这些理论逐渐有了越来越深刻的理解，时隔两年重写了这个博客（后续有新的理解会继续更新）。

# 一、抛硬币实验

假如你参与了一场赌博，你被告知一个硬币过去抛掷10次的正反情况，接下来由你下注，而你只有一次机会。这时，你会如何决策？

一般地，硬币有正反两面，如果硬币正反两面是均匀的，即每次抛掷后硬币为正的概率是 $0.5$。使用这个硬币抛10次，很可能有5次是正面。但是假如有人对硬币做了手脚，比如提前对硬币做了修改，硬币每次都会正面朝上，现在抛10次，10次都是正面，那么下次你绝对不会猜它是反面，因为前面的10次结果摆在那里，直觉上你不会相信这是一个普通的硬币。现在有一人抛了10次硬币，得到6正4反的结果，如何估算下次硬币为正的概率呢？

因为硬币并不是我们制作的，我们不了解硬币是否是完全均匀的，只能根据现在的观察结果来反推硬币的情况。假设硬币上有个参数 $\theta$，它决定了硬币的正反均匀程度， $\theta = 0.5$ 表示正反均匀，即每次抛硬币为正的概率为 $0.5$， $\theta = 1$ 表示硬币只有正面，即每次抛硬币为正的概率为1。那么，从观察到的正反结果，反推硬币的构造参数的过程 $\theta$，就是一个参数估计的过程。

##  1.1 概率函数 Probability

抛掷10次硬币可能出现不同的情况，可能是“5正5反”、“4正6反”，“10正0反”等。假如我们知道硬币是如何构造的，即已知硬币的参数，那么不同参数 $\theta$ 下，出现“6正4反”的概率为：

$$
\begin{aligned} & P(6 \text { 正 } 4 \text { 反 } \mid \theta=0.5)=C_{10}^6 \times 0.5^6 \times(1-0.5)^4 \approx 0.2051 \\ & P(6 \text { 正 } 4 \text { 反 } \mid \theta=0.6)=C_{10}^6 \times 0.6^6 \times(1-0.6)^4 \approx 0.2508 \\ & P(6 \text { 正 } 4 \text { 反 } \mid \theta=0.9)=C_{10}^6 \times 0.9^6 \times(1-0.9)^4 \approx 0.0112\end{aligned}
$$

上面这个公式是概率函数，表示已知参数 $\theta$ ，“6正4反”事件发生的概率。参数 $\theta$  取不同的值时，事件发生的概率不同。在数学上一般使用$P$ 或 $P_r$ 表示概率（Probability）函数。

上述过程中，抛10次硬币，要选出6次正面，使用了排列组合。因为“6正4反”可能会出现 `正正正正正正反反反反`、`正正正正正反正反反反`、`正正正正反正正反反反` 等共 $C_{10}^{6} = 210$ 种组合，要在10次中选出6次为正面。假如每次正面的概率是 $0.6$，那么反面的概率就是($1-0.6$)。每次抛掷硬币的动作是相互独立，互不影响的，“6正4反”发生的概率就是各次抛掷硬币的概率乘积，再乘以210种组合。

概率反映的是：**已知背后原因，推测某个结果发生的概率**。

## 1.2 似然函数 Likelihood

与概率不同，似然（Likelihood）反映的是：**已知结果，反推原因**。具体而言，似然函数表示的是基于观察的数据，取不同的参数 $\theta$ 时，统计模型以多大的可能性接近真实观察数据。这就很像开篇提到的赌局，已经给你了一系列硬币正反情况，但你并不知道硬币的构造，下次下注时你要根据已有事实，反推硬币的构造。例如，当观察到硬币“10正0反”的事实，猜测硬币极有可能每次都是正面；当观察到硬币“6正4反”的事实，猜测硬币有可能不是正反均匀的，每次出现正面的可能性是0.6。

似然函数与前面的概率函数的计算方式极其相似，与概率函数不同的是，似然函数是 $\theta$ 的函数，即 $\theta$ 是未知的。似然函数衡量的是在不同参数 $\theta$ 下，真实观察数据发生的可能性。似然函数通常是多个观测数据发生的概率的联合概率，即多个观测数据都发生的概率。

这里稍微解释一下事件独立性与联合概率之间的关系。如果事件A和事件B相互独立，那么事件A和B同时发生的概率是$ P(A) \times P(B)$。例如，事件“下雨”与事件“地面湿”就不是相互独立的，“下雨”与"地面湿"是同时发生、高度相关的，这两个事件都发生的概率就不能用概率的乘积来表示。两次抛掷硬币相互之间不影响，因此硬币正面朝上的概率可以用各次概率的乘积来表示。

似然函数通常用 $L$ 表示，对应英文 Likelihood。观察到抛硬币“6正4反”的事实，硬币参数取 $\theta$ 不同值时，似然函数表示为：

$$
L(\theta; 6 \text { 正 } 4 \text { 反 })= C_{10}^6 \times \theta ^6 \times(1-\theta )^4
$$

这个公式的图形如下图所示。从图中可以看出：参数 $\theta = 0.6$ 时，似然函数最大，参数为其他值时，“6正4反”发生的概率都相对更小。在这个赌局中，我会猜测下次硬币为正，因为根据已有观察，硬币很可能以0.6的概率为正。

![“6正4反”的似然函数](\img\Mathematics_MLE_MAP_LSM_Bayes\1.jpg)

推广到更为一般的场景，似然函数的一般形式可以用下面公式来表示，也就是之前提到的，各个样本发生的概率的乘积。

$$
L(\theta ; \mathbf{X})=P_1\left(\theta ; X_1\right) \times P_2\left(\theta ; X_2\right) \ldots \times P_n\left(\theta ; X_n\right)=\prod P_i\left(\theta ; X_i\right)
$$

> 本文公式中变量加粗表示该变量为向量或矩阵

##  1.3 最大似然估计 Maximum Likelihood Estimate (MLE)

理解了似然函数的含义，就很容易理解最大似然估计的机制。似然函数是关于模型参数的函数，是描述观察到的真实数据在不同参数下发生的概率。最大似然估计要寻找最优参数，让似然函数最大化。或者说，使用最优参数时观测数据发生的概率最大。

线性回归的误差项 $\epsilon$ 是预测值与真实值之间的差异，如下面公式所示。它可能是一些随机噪音，也可能是线性回归模型没考虑到的一些其他影响因素。

$$
y^{(i)} = \theta ^Tx^{(i)} + \epsilon^{(i)}
$$

线性回归的一大假设是：误差 $\epsilon$ 服从均值为0的正态分布，且多个观测数据之间互不影响，相互独立。正态分布（高斯分布）的概率密度公式如下面公式，根据正态分布的公式，可以得到 $\epsilon$ 概率密度。

假设 $x$ 服从正态分布，它的均值为 $\mu$ ，方差为 $\sigma ^2$ ，它的概率密度公式如下。公式左侧的 $P(x;\mu,\sigma)$ 表示 $x$ 是随机变量，分号 $;$ 强调 $\mu$ 和$\sigma$ 不是随机变量，而是这个概率密度函数的参数。条件概率函数中使用的 $\|$ 竖线有明确的意义，$P(y\|x)$ 表示给定 $x$（Given $x$），$y$ 发生的概率（Probability of $y$）。

$$
P(x ; \mu, \sigma)=\frac{1}{\sqrt{2 \pi \sigma^2}} \exp \left(-\frac{(x-\mu)^2}{2 \sigma^2}\right)
$$

误差项服从正态分布，那么：

$$
P(\epsilon^{(i)}) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(\epsilon^{(i)})^2}{2\sigma^2}) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
$$

也就是:

$$
P(y^{(i)}|x^{(i)};\theta) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
$$

上式表示给定 $x^{(i)}$，的 $y^{(i)}$概率分布。$\theta$ 并不是随机变量，而是一个参数，所以用分号 $;$ 隔开。或者说 $x^{(i)}$ 和 $\theta$ 不是同一类变量，需要分开单独理解。$P(y^{(i)}\|x^{(i)},\theta)$ 则有完全不同的意义，表示 $x^{(i)}$ 和 $\theta$ 同时发生时， $y^{(i)}$ 的概率。

前文提到，似然函数是所观察到的各个样本发生的概率的乘积。一组样本有 $m$ 个观测数据，其中单个观测数据发生的概率为刚刚得到的公式，$m$ 个观测数据的乘积如下所示。

$$
L(\theta) = L(\theta;\mathbf{X},\mathbf{Y}) = \prod P\left(y^{(i)}|x^{(i)};\theta\right)
$$

最终，似然函数可以表示成：

$$
L(\theta) = \prod^m_{i=1}\frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
$$

其中，$x^{(i)}$ 和 $y^{(i)}$ 都是观测到的真实数据，是已知的，$\theta$ 是需要去求解的模型参数。

# 二、线性回归问题的理论基础

## 2.1 最小二乘法 Least Sqaure Method (LSM)

在线性回归问题中，我们经常使用最小二乘法（又称最小平方法），将残差（即观测值和模拟值的差）的**平方和**来作为代价函数，然后使得这个代价函数的值最小，以此来得到最佳的参数 $\theta$。那么为什么会选择平方和而不是其它的指标（例如：绝对值的和或者4次方的和）呢？

直接看 **最大似然估计** 最后一个公式，现在来：找合适的参数$\theta$，使得 $L(\theta) $ 取值最大。公式两边分别取对数得到

$$
\begin{equation}
\begin{aligned}
lnL(\theta)&=ln\prod^m_{i=1}\frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}) \\
&=\sum^m_{i=1}ln\frac{1}{\sqrt{2\pi} \sigma} - \sum^m_{i=1}\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}  \\
&=m\cdot ln\frac{1}{\sqrt{2\pi} \sigma} - \frac{1}{2\sigma^2} \cdot \sum^m_{i=1}(y^{(i)}-\theta^Tx^{(i)})^2
\end{aligned}
\end{equation}
$$

若要想目标函数值最大（概率最大），那么需要$J(\theta)$最小即可。

$$
J(\theta) = \frac{1}{2}\sum^m_{i=1}(y^{(i)}-\theta^Tx^{(i)})^2
$$

这即为线性回归为何要选用最小二乘来作为衡量指标的原因，当然这个结果的前提依旧是基于 **误差 $\epsilon$ 服从均值为0的正态分布** 的假设。


## 2.2 贝叶斯定理 Bayes' Theorem

贝叶斯定理如下：

$$
P(A \cap B) = P(A)*P(B|A) = P(B)*P(A|B)
$$

$P(A \cap B)$ 是 $A$ 和 $B$ 同时发生的概率

$P(A)$ 是 $A$ 发生的概率

$P(B)$ 是 $B$ 发生的概率

$P(A\|B)$ 是给定 $B$ 的情况下，$A$ 发生的概率

$P(B\|A)$ 是给定 $A$ 的情况下，$B$ 发生的概率

该公式可变形为：

$$
P(A|B)=\frac{P(B|A)*P(A)}{P(B)}
$$

- - -

同样通过抛硬币的例子来直观地解释贝叶斯估计的思想。假如现在拿到一个硬币，不确定这个硬币是否正规（不要直接以上帝视角认为每次抛硬币正面朝上的概率都是0.5）。现在想通过抛这个硬币来估计下这个硬币被抛出去正面朝上的概率。于是我们拿这枚硬币抛了10次，得到的数据（记做事件A）是“反正正正正反正正正反（7正1反）”。我们想求的正面出现的概率$\theta$是模型参数，而抛硬币模型我们可以假设是二项分布。

那么，出现事件A，即“反正正正正反正正正反（7正1反）”的似然函数是多少呢？

$$
\begin{equation}
\begin{aligned}
L(\theta|A) 
&= (1 - \theta)*\theta*\theta*\theta*\theta*(1-\theta)*\theta*\theta*\theta*(1-\theta) \\
&= (1-\theta)^3*\theta^7
\end{aligned}
\end{equation}
$$

我们可以画出 $L(\theta \|A)$ 的图像：

```
x = 0:0.01:1; % x 即 theta
y = (1 - x).^3 .* x.^7;
plot(x,y)
hold on;
line([0.7 0.7],[0 2.5e-3], 'color', 'r') % 绘制点1(0.7,0) 和 点2(0.7,2.5e-3) 的连线
xlabel('\theta')
ylabel('likelihood')
```

![picture1](\img\Mathematics_MLE_MAP_LSM_Bayes\2.jpg "图2")

可以看出来大概是在$\theta = 0.7$左右的时候，似然最大。具体 $\theta=$多少时似然最大，可以通过对似然函数求导，令导数等于0得到。

此时，我们已经完成了对$\theta$的最大似然估计。即抛10次硬币，事件A发生，**最大似然估计学派 **认为正面向上的概率是0.7。

这个时候 **贝叶斯学派** 的人会说，硬币一般都是均匀的啊！ 就算你做实验发现结果是 事件A，我也不信硬币正面向上的概率是0.7。况且你的实验次数才10次，不足以得出准确结论。

贝叶斯学派在这里考虑了先验概率（硬币抛之前，我们知道正规的硬币正面向上的概率是0.5），根据前面提到的贝叶斯法则，后验概率可以表示为：

$$
P(\theta|A)=\frac{P(A|\theta)*P(\theta)}{P(A)}
$$

$P(A)$ 是一个已知值，所以在写公式的时候很多人会去掉了分母 $P(A)$ 。写成 $P(\theta \| A) \propto P(A \| \theta)*P(\theta)$ 。

$P(A)$为什么是一个已知值？假设“投10次硬币”是一次实验，实验做了1000次，事件A-“反正正正正反正正正反”出现了n次，则$P(A) = n/1000$

现在问题就成了 $\theta$ 在取什么值的时候， $P(\theta \| A)$ 最大，这个就叫做 **最大后验概率估计（Maximum A Posteriori Estimation, MAP）**，与 **最大似然估计（MLE）** 有异曲同工之妙。

先验概率是人为设置的，先考虑一种极端的情况，我们认为（先验的）正面朝上的概率就是0.5，那么仅仅10次的抛硬币的实验根本不会影响到我的先验知识，我依然会坚定的认为$P(\theta) = 0.5$。

但其实现实情况是，可能会存在一些劣质硬币，这里暂且给 $\theta$ 一个0.01的方差,也就是说$P(\theta) \sim N(0.5,0.1^2)$，单变量正态分布概率密度函数定义为：$P(\theta)=\frac{1}{\sqrt{2 \pi }\sigma} exp(-\frac{1}{2}\left(\frac{\theta-\mu}{\sigma}\right)^{2})$,在这里 $\mu$ 和 $\sigma$ 分别是0.5和0.1。

这时的后验概率$P(\theta \|A)$的图像为

```
x = 0:0.01:1; % x 即 theta
sigma = 0.1;
mu = 0.5;
P_theta = 1 / (sqrt(2 * pi) * sigma) * exp (- 1/2 * ((x - mu)./sigma).^2);
y = (1 - x).^3 .* x.^7 .* P_theta;
plot(x,y)
hold on;
line([0.56 0.56],[0 5e-3], 'color', 'r') % 绘制点1(0.56,0) 和 点2(0.56,5e-3) 的连线
xlabel('\theta')
ylabel('likelihood')
```

![picture3](\img\Mathematics_MLE_MAP_LSM_Bayes\3.jpg "图3")

可以看到使得后验概率最大的$\theta$的值从之前的0.7变到了0.56附近，更加往0.5靠近了。

下面对于不同的$\sigma$做多组实验

![picture4](\img\Mathematics_MLE_MAP_LSM_Bayes\4.jpg "图4")

![picture5](\img\Mathematics_MLE_MAP_LSM_Bayes\5.jpg "图5")

随着$\sigma$的增大，使得后验概率最大的 $\theta$ 会越靠近最大似然估计的结果。

至于如果准确的计算 $\sigma$ 等于多少的时候，后验概率最大，思路和最小二乘的推导方式是相通的，也是通过先求 $ln$，再求导的方式。

# 三、线性回归问题的解

## 3.1 最小二乘法的解

Cost function: 

$$
\mathcal{J}(x)=(y-f(x))^T {S}_{O}^{-1}(y-f(x))
$$

上式即2.1节最后一个公式，那么当 $\mathcal{J}(x)$ 最小时，$x$ 等于多少呢？
$$
f(x) = K x \\
\mathcal{J}(x)=(K x - y)^T {S}_{O}^{-1}(K x - y) \\
\text { 记： } V = K x - y \\
\text { 解 }\frac{\partial{V^T {S}_{O}^{-1} V}}{\partial{x} } = 0 \text { 时的 } x\\
\frac{\partial{V^T {S}_{O}^{-1} V}}{\partial{x} } = 2 V^T {S}_{O}^{-1} \frac{\partial{V}}{\partial{x}} \\
V^T {S}_{O}^{-1} K = 0 \\
\text { 上式转置: } K^T {S}_{O}^{-1} V = 0 \\
K^T {S}_{O}^{-1} (Kx - y) = 0 \\
K^T {S}_{O}^{-1} K x - K^T {S}_{O}^{-1} y = 0 \\
x = (K^T{S}_{O}^{-1} K)^{-1}K^T {S}_{O}^{-1} y
$$


Solution: 
$$
\hat{x}=(K^T{S}_{O}^{-1} K)^{-1}K^T {S}_{O}^{-1} y
$$

## 3.2 贝叶斯估计的解

Cost function: 

$$
\mathcal{J}(x)= \frac{1}{2}(y-f(x))^T {S}_{O}^{-1}(y-f(x))+\frac{1}{2}(x-x_a)^T {S}_{a}^{-1}(x-x_a)
$$

Solution 1(观测个数大于未知数个数时，这种形式计算效率更高): 

$$
x_p={x}_{a}+(K^T{S}_{O}^{-1} K + {S}_{a}^{-1})^{-1}K^T {S}_{O}^{-1} (y-K{x}_{a})
$$

Solution 2(观测个数小于未知数个数时，这种形式计算效率更高): 

$$
x_p={x}_{a}+(K S_a)^T (K{S}_{a}^{-1} K^T + {S}_{o})^{-1} (y-K x_a)
$$

Where $K$ is the $m\times n$ Jacobian matrix, $S_O$ is the $m\times m$ observational error covariance matrix, $S_a$ is the $n \times n$ prior error covariance matrix, $y$ is the $m \times1$ observation vector, $x$ is the $n \times1$ state vector, $x_a$ is the  $n \times1$ prior state vector.

Posterior error covariance:

$$
S_p=(K^T{S}_{O}^{-1} K + {S}_{a}^{-1})^{-1}
$$

Averaging kernel:

$$
A=\partial{x_p} / \partial{\bar{x}} \\
x_p={x}_{a}+(K^T{S}_{O}^{-1} K + {S}_{a}^{-1})^{-1}K^T {S}_{O}^{-1} (K \bar{x} -K{x}_{a})\\
x_p={x}_{a}+(K^T{S}_{O}^{-1} K + {S}_{a}^{-1})^{-1}K^T {S}_{O}^{-1} K(\bar{x} - {x}_{a})\\
A = (K^T{S}_{O}^{-1} K + {S}_{a}^{-1})^{-1} K^T{S}_{O}^{-1} K
$$



# 参考链接

[最小二乘的概率解释_什么是最小二乘回归(least squares regression),写出它的损失函数,试用从概率-CSDN博客](https://blog.csdn.net/acdreamers/article/details/44662823)

[一文搞懂极大似然估计 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/26614750)

[最大似然估计，鲁老师 (lulaoshi.info)](https://lulaoshi.info/deep-learning/linear-model/maximum-likelihood-estimation.html)

[详解最大似然估计（MLE）、最大后验概率估计（MAP），以及贝叶斯公式的理解-CSDN博客](https://blog.csdn.net/u011508640/article/details/72815981)

[PRML笔记1-从贝叶斯先验的角度解释正则化 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/270545697)