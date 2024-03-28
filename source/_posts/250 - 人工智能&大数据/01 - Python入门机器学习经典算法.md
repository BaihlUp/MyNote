---
title: Python入门机器学习经典算法
date: 2023-09-18 16:27:41
categories:
  - 人工智能&大数据
tags:
  - 机器学习
published: false
---





Anaconda 是一个方便的python包管理和环境管理软件

Jupyter notebook

基础工具包：
Panda、Numpy、Matplolib


安装 jupyterthemes


# 1 机器学习基本概念

分类问题：
回归问题：
监督学习：给机器的训练数据拥有“标记”或者“答案”
	监督类学习算法：K近邻，线性回归和多项式回归、逻辑回归、SVM、决策树和随机森林

非监督学习：对没有“标记”的数据进行分类 -- 聚类分析
非监督学习的意义：对数据进行降维处理
	特征提取：信用卡的信用评级和人的胖瘦无关
	特征压缩：尽量缩小损失的情况下，尽量把多维特征的向量压缩，如：PCA
	异常检测：

半监督学习：一部分数据有“标记”或者“答案”，另一部分数据没有

增强学习：根据周围环境的情况，采取行动，根据采取行动的结果，学习行动方式。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231210183041.png)


批量学习：
在线学习：对数据进行在线实时学习。
	优点：及时反映新的环境变化
	问题：新的数据带来不好的变化
	解决方案：需要加强对数据进行监控
	一些应用：例如适用于数据量巨大，完全无法批量学习的环境。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231210183621.png)

参数学习：
	一旦学到了参数，就不再需要原有的数据集


# 2 最基础的分类算法-k近邻算法 kNN


- **欧拉距离**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231210194323.png)

上图中是计算二维到多维的欧拉距离，标识两个点之间的距离。

kNN算法是通过计算欧拉距离，寻找给定的数据，与已有数据集那些最接近，从而实现预测。

KNN测试程序：
![[Pasted image 20231212212559.png]]

![[Pasted image 20231212212621.png]]

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231212212742.png)

K 近邻算法非常特殊，可以被认为没有模型的算法

使用 scikit-learn 中的 kNN：
![[Pasted image 20231212213127.png]]

以上有告警，是因为 predict函数中传入的参数最好是一个矩阵，下边把 x 转换成矩阵：

![[Pasted image 20231212213235.png]]

封装后的 kNN 类代码：kNN.py

调用上边的kNN.py：

![[Pasted image 20231212214039.png]]


## 判断机器学习算法的性能

1. 把数据集分为 训练数据集 和 测试数据，通过测试数据直接判断模型好坏，在模型进入真是环境前改进模型。

测试算法示例：

使用 sklearn中自带的数据集：
![[Pasted image 20231212220920.png]]

对 x，y的数据集进行乱序处理，因为x，y是一一对应的关系，如果x要乱序，y对应的元素也需要乱序，使用如下方法：
![[Pasted image 20231212221228.png]]
![[Pasted image 20231212221333.png]]

以上示例对训练数据集和测试数据集分离。测试数据集占 20%

具体代码实现：model_selection.py
引用 model_selection.py中的算法，看下算法的准确度
![[Pasted image 20231212222051.png]]


![[Pasted image 20231212221957.png]]


## 分类准确度



