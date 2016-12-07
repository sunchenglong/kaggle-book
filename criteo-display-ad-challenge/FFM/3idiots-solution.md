# 比赛排名第一的解法（来自3idiots）

这篇文章主要翻译于作者的PPT。

## 整理资料来源

* kaggle论坛

  https://www.kaggle.com/c/criteo-display-ad-challenge/forums/t/10555/3-idiots-solution-libffm

  * 实现方法的PPT

  http://www.csie.ntu.edu.tw/~r01922136/kaggle-2014-criteo.pdf

* 实现的代码

  https://github.com/guestwalk/kaggle-2014-criteo

* libffm工具包

  http://www.csie.ntu.edu.tw/~r01922136/libffm/

* libffm 源码

  https://github.com/guestwalk/libffm

## 作者

YuChin Juan, Yong Zhuang, and Wei-Sheng Chin

NTU CSIE MLGroup

在这次比赛中，在公共排行榜(public leaderboards)的分数为0.44488在私有排行榜(private leaderboards)上的排名是0.44479。


## 实现概述

数据总共有13列数值特征，26列类别特征(hashcode),作者首先将26列类别特征放入gbdt中训练30棵高度均为7的树，每棵树作为一个特征，共有30个特征，特征的值是最后显现出值的叶子节点的序号，即这个值为0-255，作者最后将13+26+30一共69个特征的标签经过hash处理，然后与10^6取模做为特征的索引，值仍用原来特征的值，获取到10^6个one-hot编码后的稀疏特征矩阵，放入FFM模型中进行训练。


## 小规模数据

![img](../img/3idiotsDataset.png)

作者为了演示算法的效果，采样了一份小数据集，可以看到如之前所述，数据第一列为标签，1代表点击，0代表未点击，最后？表示需要预测0-1的一个点击的概率值，I1-I13表示。。。//Todo

## 技术路线

![img](../img/3idiot.png)

总体的实现方案如上图所示，CSV表示输入的数据文件，
Pre-A

目标是：为GBDT生成特征
* 包括所有数值型特征(13个特征)
* 类型特征(如果在one-hot编码后出现超过400万个特征)(26个特征)




GBDT

目标：生成GBDT特征
* 在GBDT中使用树生成特征
* 使用的树是深度为7的30个树
* 生成的是这30棵树每棵最显著的特征，也就是生成了30个特征
* 这个方法是Xinran He et al发表的(Facebook).

Purpose: generate GBDT features.
• We use trees in GBDT to generate features.
• 30 trees with depth 7 are used.
• 30 features are generated for each impression.
• This approach is proposed by  at Facebook.
• The imlementation of GBDT is base on Algorithm 5 in the
following slides:
http://statweb.stanford.edu/ ~ jhf/ftp/trebst.pdf


Pre-B

Purpose: generate features for FFM.
• Numerical features (I1-I13) greater than 2 are transformed by
v ← blog(v) 2 c.
• Categorical features (C1-C26) appear less than 10 times are
transformed into a sepcial value.
• GBDT features are directly included.
• These three groups of features are hashed into 1M-dimension
by hashing trick.
• Each impression has 13 (numerical) + 26 (categorical) + 30
(GBDT) = 69 features.

Calib(calibrate)表示标准化


Rst(result)最后的输出文件

## 特征工程




## 多算法对比实验

![img](../img/3idiotExperiment.png)

## 实验运行情况

双6核CPU环境下的运行状况，所有的进程都是并行的
![img](../img/3idiotRuntime.png)

## FFM算法
