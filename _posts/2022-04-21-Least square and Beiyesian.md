---
layout:     post
title:      最小二乗法和贝叶斯估计
subtitle:   从概率角度看最小二乗法（least sqaure method）和贝叶斯估计
date:       2022-04-21
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
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
P(y^{(i)}|x^{(i)};\sigma) = \frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}) 
$$

即
$$
y^{(i)}|x^{(i)};\theta \sim N(\theta^Tx^{(i)},\sigma^2)
$$

进一步得到联合概率密度函数

$$
L(\theta) = P(Y|X;\theta) = \prod^m_{i=1}\frac{1}{\sqrt{2\pi} \sigma}exp(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
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



最小二乘的概率解释
https://blog.csdn.net/acdreamers/article/details/44662823

极大似然估计
https://zhuanlan.zhihu.com/p/26614750

最大后验概率估计
https://blog.csdn.net/u011508640/article/details/72815981
