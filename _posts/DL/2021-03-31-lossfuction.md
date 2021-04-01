---
layout: single
title: "SoftMax和CrossEntropy"
toc: true
toc_label: "目录"
toc_icon: "bars"
category: 深度学习基础
date: 2021-03-31 10:00+0000
typora-root-url: ../../../blog
---

#### 二分类问题

对于二分类问题，常见的算法是逻辑回归，逻辑回归使用Sigmoid激活函数来将输出映射到$[0,1]$之间，以表示某类事件发生的概率。
$$
S(x)=\frac{1}{1+e^{-x}}
$$
![Sigmoid 曲线](/assets/images/sigmoid.png)

使用逻辑回归算法预测二分类问题时，可以在输出层设置一个节点，用来表示某个事件A发生的概率$P(A|x)$；也可以设置两个节点来表示事件$P(A|x)$和$P(\bar{A}|x)$发生的概率，满足条件$P(A|x)+P(\bar{A}|x)=1$，类似地也可以推广到n个输出节点。这样相比二分类问题就多出了个约束条件：概率相加为1。于是我们就希望有一种函数不仅可以将输出映射到$[0,1]$并且自然地满足概率相加为1的条件，这个函数就是SoftMax函数。



#### SoftMax函数

SoftMax的含义在于，它不是唯一的确定一个max值，而是通过给定概率来表示每个输出类别发生的可能性，概率大的类别更可能发生，但并不代表概率小的类别一定不会发生。其数学定义如下：
$$
\operatorname{Softmax}\left(z_{i}\right)=\frac{e^{z_{i}}}{\sum_{c=1}^{C} e^{z_{c}}}
$$
其中$z_i$为第$i$个节点的输出值，C为输出类别数，通过上式就能将输出限制在$[0,1]$之间，且满足概率和为1.

#### SoftMax的优缺点

- 优点：
  - 通过引入指数函数，可以使得输出对输入的敏感度增强，可以快速的拉开不同类别之间的距离
  - 指数函数求导方便
- 缺点：
  
  - 指数爆炸容易造成参数溢出
  
  > 因此通常使用logsoftmax来代替softmax，使得在数值上相对稳定些
  > $$
  > \begin{array}{c}
  > \log \left(S_{j}\right)=\log \left(\frac{e^{a_{j}}}{\sum_{k=1}^{N} e^{a_{k}}}\right) \\
  > =\log \left(e^{a_{j}}\right)-\log \left(\sum_{k=1}^{N} e^{a_{k}}\right) \\
  > =a_{j}-\log \left(\sum_{k=1}^{N} e^{a_{k}}\right)
  > \end{array}
  > $$

#### CrossEntropy交叉熵损失

SoftMax通常和交叉熵损失结合使用，交叉熵是用来衡量两个概率分布之间的距离的，两个分布距离越近，交叉熵的值就越小，它的公式是：
$$
L=-\sum_{c=1}^{C} y_{c} \log \left(p_{c}\right)
$$
将softmax与交叉熵结合起来，这里的$y_c$是正确的标签值，通常为one-hot编码，即只有为真的那一位是1，其他位都是0，$p_c$就是softmax输出的概率，取log后正好可以保证一定的数值稳定性。

将二者结合后的梯度计算结果非常精简，
$$
\frac{\partial L}{\partial z_{i}}=p_{i}-y_{i}
$$
$Loss$对上层输入$z_i$的梯度就等于，在正确标签的那一位上用softmax输出的概率值减去正确的$y$值（即1），其他位的值就是softmax输出的概率值（因为$y$为0）。

更新参数时（梯度的负方向），可以总结为：

1. 先将所有的 ![[公式]](https://www.zhihu.com/equation?tex=z) 值减去对应的Softmax的结果，记为推所有；
2. 然后将真实标记中的对应位置的值加上1，记为拉一个；