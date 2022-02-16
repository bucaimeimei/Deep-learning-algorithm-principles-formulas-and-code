1、xgboost是什么
------

全称：eXtreme Gradient Boosting 

作者：陈天奇(华盛顿大学博士) 

基础：GBDT 

所属：boosting迭代型、树类算法。 

适用范围：分类、回归 

优点：速度快、效果好、能处理大规模数据、支持多种语言、支 持自定义损失函数等等。 

缺点：发布时间短（2014），工业领域应用较少，待检验


2、基础知识，GBDT
--------

xgboost是在GBDT的基础上对boosting算法进行的改进，内部决策树使用的是回归树，简单回顾GBDT如下： 



回归树的分裂结点对于平方损失函数，拟合的就是残差；对于一般损失函数（梯度下降），拟合的就是残差的近似值，分裂结点划分时枚举所有特征的值，选取划分点。 

最后预测的结果是每棵树的预测结果相加。


3、xgboost算法原理知识
--------

3.1 定义树的复杂度
------



把树拆分成结构部分q和叶子权重部分w。 

树的复杂度函数和样例： 


定义树的结构和复杂度的原因很简单，这样就可以衡量模型的复杂度了啊，从而可以有效控制过拟合。


3.2 xgboost中的boosting tree模型
-----------


和传统的boosting tree模型一样，xgboost的提升模型也是采用的残差（或梯度负方向），不同的是分裂结点选取的时候不一定是最小平方损失。 



3.3 对目标函数的改写 
-------


最终的目标函数只依赖于每个数据点的在误差函数上的一阶导数和二阶导数。这么写的原因很明显，由于之前的目标函数求最优解的过程中只对平方损失函数时候方便求，对于其他的损失函数变得很复杂，通过二阶泰勒展开式的变换，这样求解其他损失函数变得可行了。很赞！ 

当定义了分裂候选集合的时候，可以进一步改目标函数。分裂结点的候选响集是很关键的一步，这是xgboost速度快的保证，怎么选出来这个集合，后面会介绍。 


求解： 



3.4 树结构的打分函数 
-------
Obj代表了当指定一个树的结构的时候，在目标上面最多减少多少。(structure score)



对于每一次尝试去对已有的叶子加入一个分割 

 
这样就可以在建树的过程中动态的选择是否要添加一个结点。 

 
假设要枚举所有x < a 这样的条件，对于某个特定的分割a，要计算a左边和右边的导数和。对于所有的a，我们只要做一遍从左到右的扫描就可以枚举出所有分割的梯度和GL、GR。然后用上面的公式计算每个分割方案的分数就可以了。


3.5 寻找分裂结点的候选集 
-------
1、暴力枚举


2、近似方法 ，近似方法通过特征的分布，按照百分比确定一组候选分裂点，通过遍历所有的候选分裂点来找到最佳分裂点。 

两种策略：全局策略和局部策略。在全局策略中，对每一个特征确定一个全局的候选分裂点集合，就不再改变；而在局部策略中，每一次分裂 都要重选一次分裂点。前者需要较大的分裂集合，后者可以小一点。对比补充候选集策略与分裂点数目对模型的影响。 全局策略需要更细的分裂点才能和局部策略差不多


3、Weighted Quantile Sketch



陈天奇提出并从概率角度证明了一种带权重的分布式的Quantile Sketch。


4、xgboost的改进点总结
------

1、目标函数通过二阶泰勒展开式做近似 

2、定义了树的复杂度，并应用到目标函数中 

3、分裂结点处通过结构打分和分割损失动态生长 

4、分裂结点的候选集合通过一种分布式Quantile Sketch得到 

5、可以处理稀疏、缺失数据 

6、可以通过特征的列采样防止过拟合


5、参数
----

xgboost 有很多可调参数，具有极大的自定义灵活性。比如说： 

（1）objective [ default=reg:linear ] 定义学习任务及相应的学习目标，可选的目标函数如下： 

“reg:linear” –线性回归。 

“reg:logistic” –逻辑回归。 

“binary:logistic” –二分类的逻辑回归问题，输出为概率。 

“multi:softmax” –处理多分类问题，同时需要设置参数num_class（类别个数） 

（2）’eval_metric’ The choices are listed below，评估指标: 

“rmse”: root mean square error 

“logloss”: negative log-likelihood 

(3)max_depth [default=6] 数的最大深度。缺省值为6 ，取值范围为：[1,∞]


参考： 

官方文档：

http://xgboost.readthedocs.io/en/latest/


Github： 

https://github.com/dmlc/xgboost


Xgboost论文： 

http://cran.fhcrc.org/web/packages/xgboost/vignettes/xgboost.pdf


陈天奇的boosting tree的ppt： 

http://homes.cs.washington.edu/~tqchen/pdf/BoostedTree.pdf


Xgboost调参： 

http://blog.csdn.net/wzmsltw/article/details/50994481


GBDT资料： 

http://www.jianshu.com/p/005a4e6ac775
