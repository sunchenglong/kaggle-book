# DSP模型中FM FFM模型

机器学习方法应用在DSP广告投放中，预估CTR/CVR，业界常用的方法是人工特征工程+LR,GBDT,近期FM,FFM模型在其中使用效果非常显著。

先解释一波DSP相关专业术语：
DSP:Demand-Side platform，需求方平台，在互联网广告产业中，DSP是一个系统，也是一种在线广告平台，服务于广告主，帮助广告主在互联网上进行广告投放。两个核心特征：强大的RTB(Real-Time Bidding)的基础设施和能力，强大的用户定位(Audience Targeting)技术。
CPC:Cost Per Click按点击计费
CPA:Cost per Action按成果数计费
CTR: Click Through Rate 点击率
CVR: Click Value Rate 转化率，衡量CPA广告效果的指标。
了解DSP中的基础算法和模型，可以参考M6D公司的文章，以及一份中文解释：http://www.techinads.com/archives/41

FM:Factorization Machine 因子分解机，又称分解机器，旨在解决大规模稀疏数据下的特征组合问题。
FFM:Field-aware Factorization Machine,场感知分解机器，把相同性质的特征归于同一个field

稀疏问题引出：
在机器学习中，尤其是计算广告领域，特征并不总是数值型，很多时候是分类值，对于categorical feature，通常会采用one-hot encoding转换成数值型特征，转化过程会产生大量稀疏数据。

one-hot encoding:独热编码或者叫一位有效编码，使用N位状态寄存器来对N个状态进行编码，每个状态都有它独立的寄存器，并且在任意时刻，只有一位有效。可以这么理解：对于每一个特征，如果它有m个可能取值，那么经过one-hot encoding之后，就变成了m个二元特征，并且，这些特征互斥，每次只有一个激活，因此，数据会变得稀疏。

好处：1.解决了分类器不好处理属性数据的问题。2.在一定程度上起到了扩充特征的作用。
