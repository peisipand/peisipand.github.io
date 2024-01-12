---
layout:     post
title:      最小二乘法和贝叶斯估计
subtitle:   从概率角度看最小二乘法（least sqaure method）和贝叶斯估计
date:       2022-04-21
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Mathematics
---



# 1. 最小二乘
在线性回归中，我们以残差（观测和模拟差）的平方和来作为损失函数，然后使得这个损失函数的值最小，以此来得到最佳的参数$\theta$。那么为什么会选择平方和而不是其它的指标（例如：绝对值的和或者4次方的和）呢？

下面从概率的角度来看一看。首先，设

$$
y^{(i)} = \theta^Tx^{(i)} + \epsilon^{(i)}
$$

误差项$\epsilon^{(i)}$服从高斯分布 $\epsilon^{(i)} \sim N(0,\sigma^2)$。由于$x^{(i)}$和$y^{(i)}$是给定的，$\theta$和$\epsilon^{(i)}$是对应的，也就是确定了$\theta$，$\epsilon^{(i)}$就确定了。$\epsilon^{(i)}$概率密度函数为

$$
P(\epsilon^{(i)}) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(\epsilon^{(i)})^2}{2\sigma^2}) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
$$

也就是

$$
P(y^{(i)}|x^{(i)};\theta) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}) 
$$

即

$$
y^{(i)}|x^{(i)};\theta \sim N(\theta^Tx^{(i)},\sigma^2)
$$


进一步得到联合概率密度函数:

$$
P(Y|X;\theta) = \prod^m_{i=1}\frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
$$

- - -
对于函数：$P(x|\theta)$。输入有两个：$x$表示某一个具体的数据；$\theta$表示模型的参数。

如果$\theta$是已知确定的，$x$是变量，这个函数叫做概率函数(probability function)，它描述对于不同的样本点$x$，其出现概率是多少。

如果$x$是已知确定的，$\theta$是变量，这个函数叫做似然函数(likelihood function), 它描述对于不同的模型参数$\theta$，出现x这个样本点的概率是多少。

给定输出$x$时，关于参数θ的似然函数$L(\theta|x)$（在数值上）等于给定参数θ后变量$x$出现的概率$P(x|\theta)$。
- - -

似然函数为：

$$
L(\theta) = P(Y|X;\theta)
$$


现在来求最大似然估计，即找到合适的参数$\theta$，使得上述概率取值最大。两边分别取对数得到

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

这即为线性回归为何要选用最小二乘来作为衡量指标的原因。


# 2.贝叶斯估计

首先来看下贝叶斯法则

- - -

$$
P(A \cap B) = P(A)*P(B|A) = P(B)*P(A|B)
$$

该公式也可变形为：

$$
P(A|B)=\frac{P(B|A)*P(A)}{P(B)}
$$

- - -

同样通过抛硬币的例子来直观地解释贝叶斯估计的思想。假如现在拿到一个硬币，不确定这个硬币是否正规（不要直接以上帝视角认为每次抛硬币正面朝上的概率都是0.5）。现在想通过抛这个硬币来估计下这个硬币被抛出去正面朝上的概率。于是我们拿这枚硬币抛了10次，得到的数据（记做事件A）是“反正正正正反正正正反”。我们想求的正面出现的概率$\theta$是模型参数，而抛硬币模型我们可以假设是二项分布。
那么，出现实验结果(事件A)（即反正正正正反正正正反）的似然函数是多少呢？

$$
\begin{equation}
\begin{aligned}
L(\theta|A) 
&= (1 - \theta)*\theta*\theta*\theta*\theta*(1-\theta)*\theta*\theta*\theta*(1-\theta) \\
&= (1-\theta)^3*\theta^7
\end{aligned}
\end{equation}
$$

我们可以画出$L(\theta|A)$的图像:
![picture1](/img/Mathematics_Beyes/1.jpg "图1")

可以看出来大概是在$\theta = 0.7$左右的时候，似然最大。（具体$\theta=$多少时似然最大，可以通过对似然函数求导，令导数等于0得到。）

此时，我们已经完成了对$\theta$的最大似然估计。即抛10次硬币，事件A发生，最大似然估计认为正面向上的概率是0.7。

这个时候贝叶斯学派的人会说，硬币一般都是均匀的啊！ 就算你做实验发现结果是 事件A，我也不信硬币正面向上的概率是0.7。况且你的实验次数才10次，不足以得出准确结论。

贝叶斯学派在这里考虑了先验概率（硬币抛之前，我们知道正规的硬币正面向上的概率是0.5），根据前面提到的贝叶斯法则,后验概率可以表示为

$$
P(\theta|A)=\frac{P(A|\theta)*P(\theta)}{P(A)}
$$

$P(A)$ 是一个已知值，所以在写公式的时候很多人会去掉了分母 $P(A)$ 。写成 $P(\theta \| A) \propto P(A \| \theta)*P(\theta)$ 。

($P(A)$为什么是一个已知值：假设“投10次硬币”是一次实验，实验做了1000次，事件A-“反正正正正反正正正反”出现了n次，则$P(A) = n/1000$）

现在问题就成了 $\theta$ 在取什么值的时候， $P(\theta \| A)$ 最大，即 最大后验概率估计（MAP），与 最大似然估计（MLE） 有异曲同工之妙。

先验概率是人为设置的，先考虑一种极端的情况，我们认为（先验的）正面朝上的概率就是0.5，那么仅仅10次的抛硬币的实验根本不会影响到我的先验知识，我依然会坚定的认为$P(\theta) = 0.5$。

但其实现实情况是，可能会存在一些劣质硬币，这里暂且给$\theta$一个0.01的方差,也就是说$P(\theta) \sim N(0.5,0.01)$，单变量正态分布概率密度函数定义为：$P(\theta)=\frac{1}{\sqrt{2 \pi }\sigma} exp(-\frac{1}{2}\left(\frac{\theta-\mu}{\sigma}\right)^{2})$,在这里$\mu$和$\sigma$分别是0.5和0.1。

这时的后验概率$P(\theta|A)$的图像为
![picture1](/img/Mathematics_Beyes/2.jpg "图2")

可以看到使得后验概率最大的$\theta$的值从之前的0.7变到了0.561，更加往0.5靠近了。

下面对于不同的$\sigma$做多组实验
![picture1](/img/Mathematics_Beyes/3.jpg "图3")
![picture1](/img/Mathematics_Beyes/4.jpg "图4")

随着$\sigma$的增大，我们得到的$\theta$（使得最大后验概率最大）会越靠近最大似然估计的结果。

至于如果准确的计算$\sigma$等于多少的时候，后验概率最大，思路和最小二乘的推导方式是相通的，也是通过先求ln，再求导的方式。下面一节会介绍。







# 参考链接

最小二乘的概率解释
https://blog.csdn.net/acdreamers/article/details/44662823

极大似然估计
https://zhuanlan.zhihu.com/p/26614750

最大后验概率估计
https://blog.csdn.net/u011508640/article/details/72815981

贝叶斯
https://zhuanlan.zhihu.com/p/270545697