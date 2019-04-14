---
layout:     post
title:      "one-pixel attack in CIFAR-10 (Keras)"
subtitle:   " \"跑该论文的实验，弄懂代码，并运用在自己的数据上\""
date:       2019-04-14 13:00:00
author:     "sjp"
header-img: "img/post-bg-forest.jpg"
catalog: true
tags:
    - 对抗样本
    - 论文实验笔记
---



# one-pixel attack in CIFAR-10 (Keras)

### 1. 一些函数的含义
**from keras import backend as K**：导入后端模块

Keras 是一个模型级库，为开发深度学习模型提供了高层次的构建模块。它不处理诸如张量乘积和卷积等低级操作。相反，它依赖于一个专门的、优化的张量操作库来完成这个操作，它可以作为 Keras 的「后端引擎」。相比单独地选择一个张量库，而将 Keras 的实现与该库相关联，Keras 以模块方式处理这个问题，并且可以将几个不同的后端引擎无缝嵌入到 Keras 中。

目前，Keras 有三个后端实现可用：TensorFlow 后端，Theano 后端，CNTK 后端。

- TensorFlow - 是由 Google 开发的一个开源符号级张量操作框架。

- Theano - 是由蒙特利尔大学的 LISA Lab 开发的一个开源符号级张量操作框架。

- CNTK - 是由微软开发的一个深度学习开源工具包。

在 $HOME/.keras/keras.json 找到 Keras 配置文件，字段 **backend** 说明了默认使用的是哪一个后端。如果你希望你编写的 Keras 模块与 Theano (th) 或 TensorFlow (tf) 等兼容，则必须通过抽象 Keras 后端 API 来编写它们。

![](https://ws1.sinaimg.cn/large/95bff755ly1g21zxnoyonj20le07hdg7.jpg)

我这里相当于使用了 tf 后端，则当我想使用 tf.placeholder() 或 tf.Variable()时，可用 K.placeholder() 和 K.variable() 代替。

----
**xs.ndim**：ndarray的维度的个数，即shape属性的长度

一维 ndim=1，二维 ndim=2，三维 ndim=3 ....

![](https://ws1.sinaimg.cn/large/95bff755ly1g220qc2tamj20vz0hm754.jpg)

-----
**np.astype()**：转换数据类型

----
**np.tile(A, reps)**：A是数组，reps 是 list

输入：reps 的元素表示对A的各个 axis 进行重复的次数
返回：一个数组，维度的数量等于 max(A.ndim, len(reps))，注意不要混淆 A.ndim 和 A.shape

有两种特殊情况：

若A.ndim < len(reps)，此时需要调整A的维度使得 A.ndim = len(reps)，即添加长度为1的维度，注意:新的维度在原维度的前面，比如原来的 A.shape 是(3, 5)，调整后是(1, 3, 5)

若A.ndim > len(reps)，此时需要增加 list 的长度，使得   A.ndim = len(reps)，即在 reps 的最前面增加元素1，比如原来的list是[2, 2]，增加长度后是[1, 2, 2]

```python
# 示例1,正常情况
a = np.array([0, 1, 2])
# 将axis=0重复2次
np.tile(A=a, reps=2)
# array([0, 1, 2, 0, 1, 2])

# 示例2,特殊情况1:A.ndim < len(reps)
a = np.array([0, 1, 2])
# 将a.shape调整至(1,3),然后将axis=0重复2次,将axis=1重复2次
np.tile(A=a, reps=(2, 2))
#array([[0, 1, 2, 0, 1, 2], [0, 1, 2, 0, 1, 2]])

# 示例3,特殊情况1:A.ndim < len(reps)
a = np.array([0, 1, 2])
# 将a.shape调整至(1,1,3),然后将axis=0重复2次,将axis=1重复1次,将axis=2重复2次
np.tile(A=a, reps=(2, 1, 2))
#array([[[0, 1, 2, 0, 1, 2]], [[0, 1, 2, 0, 1, 2]]])

# 示例4,特殊情况2:A.ndim > len(reps)
b = np.array([[1, 2], [3, 4]])
# 将reps=[2]调整至[1,2],然后将axis=0重复1次,将axis=1重复2次
np.tile(A=b, reps=2)
#array([[1, 2, 1, 2], [3, 4, 3, 4]])

# 示例5,正常情况
b = np.array([[1, 2], [3, 4]])
# 将axis=0重复两次,将axis=1重复1次
np.tile(A=b, reps=(2, 1))
#array([[1, 2], [3, 4], [1, 2], [3, 4]])

# 示例6,特殊情况1:A.ndim < len(reps)
c = np.array([1,2,3,4])
# 将c.shape调整至(4,1),然后将axis=0重复4次,将axis=1重复1次
np.tile(A=c, reps=(4,1))
# array([[1, 2, 3, 4], [1, 2, 3, 4], [1, 2, 3, 4], [1, 2, 3, 4]])
```








参考博文：

https://blog.csdn.net/littlehaes/article/details/83472225 

