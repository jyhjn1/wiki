---
title: "xgboost参数和调参技巧"
layout: page
date: 2018-08-14 10:00
---
# 写在前面
对于小样本数据,传统的机器学习方法效果可能会比深度学习好

# xgboost参数
XGBoost的参数可以分为三种类型：**通用参数**、**booster参数**以及**学习目标参数**

- General parameters：参数控制在提升（boosting）过程中使用哪种booster，常用的booster有树模型（tree）和线性模型（linear model）。
- Booster parameters：这取决于使用哪种booster。
- Learning Task parameters：控制学习的场景，例如在回归问题中会使用不同的参数控制排序。
除了以上参数还可能有其它参数，在命令行中使用

### General Parameters

- booster [default=gbtree] 

有两种模型可以选择gbtree和gblinear。gbtree使用基于树的模型进行提升计算，gblinear使用线性模型进行提升计算。缺省值为gbtree

- silent [default=0] 

取0时表示打印出运行时信息，取1时表示以缄默方式运行，不打印运行时的信息。缺省值为0
建议取0，过程中的输出数据有助于理解模型以及调参。另外实际上我设置其为1也通常无法缄默运行。。

- nthread [default to maximum number of threads available if not set] 

XGBoost运行时的线程数。缺省值是当前系统可以获得的最大线程数
如果你希望以最大速度运行，建议不设置这个参数，模型将自动获得最大线程

- num_pbuffer [set automatically by xgboost, no need to be set by user] 

size of prediction buffer, normally set to number of training instances. The buffers are used to save the prediction results of last boosting step.

- num_feature [set automatically by xgboost, no need to be set by user] 

boosting过程中用到的特征维数，设置为特征个数。XGBoost会自动设置，不需要手工设置

### Booster Parameters

From xgboost-unity, the bst: prefix is no longer needed for booster parameters. Parameter with or without bst: prefix will be equivalent(i.e. both bst:eta and eta will be valid parameter setting) .

Parameter for Tree Booster

- eta [default=0.3] 

为了防止过拟合，更新过程中用到的收缩步长。在每次提升计算之后，算法会直接获得新特征的权重。 eta通过缩减特征的权重使提升计算过程更加保守。缺省值为0.3
取值范围为：[0,1]
通常最后设置eta为0.01~0.2

- gamma [default=0] 

minimum loss reduction required to make a further partition on a leaf node of the tree. the larger, the more conservative the algorithm will be.

- range: [0,∞]

模型在默认情况下，对于一个节点的划分只有在其loss function 得到结果大于0的情况下才进行，而- gamma 给定了所需的最低loss function的值

gamma值使得算法更conservation，且其值依赖于loss function ，在模型中应该进行调参。

- max_depth [default=6] 

树的最大深度。缺省值为6
取值范围为：[1,∞]
指树的最大深度
树的深度越大，则对数据的拟合程度越高（过拟合程度也越高）。即该参数也是控制过拟合
建议通过交叉验证（xgb.cv ) 进行调参
通常取值：3-10

- min_child_weight [default=1] 

孩子节点中最小的样本权重和。如果一个叶子节点的样本权重和小于min_child_weight则拆分过程结束。在现行回归模型中，这个参数是指建立每个模型所需要的最小样本数。该成熟越大算法越

- conservative。即调大这个参数能够控制过拟合。
取值范围为: [0,∞]

- max_delta_step [default=0] 

Maximum delta step we allow each tree’s weight estimation to be. If the value is set to 0, it means there is no constraint. If it is set to a positive value, it can help making the update step more conservative. Usually this parameter is not needed, but it might help in logistic regression when class is extremely imbalanced. Set it to value of 1-10 might help control the update

取值范围为：[0,∞]

如果取值为0，那么意味着无限制。如果取为正数，则其使得xgboost更新过程更加保守。
通常不需要设置这个值，但在使用logistics 回归时，若类别极度不平衡，则调整该参数可能有效果

- subsample [default=1] 

用于训练模型的子样本占整个样本集合的比例。如果设置为0.5则意味着XGBoost将随机的从整个样本集合中抽取出50%的子样本建立树模型，这能够防止过拟合。

取值范围为：(0,1]

- colsample_bytree [default=1] 

在建立树时对特征随机采样的比例。缺省值为1

取值范围：(0,1]

- colsample_bylevel[default=1]

决定每次节点划分时子样例的比例
通常不使用，因为subsample和colsample_bytree已经可以起到相同的作用了

- scale_pos_weight[default=0]

A value greater than 0 can be used in case of high class imbalance as it helps in faster convergence.

大于0的取值可以处理类别不平衡的情况。帮助模型更快收敛

### Parameter for Linear Booster

- lambda [default=0] 

L2 正则的惩罚系数
用于处理XGBoost的正则化部分。通常不使用，但可以用来降低过拟合

- alpha [default=0] 

L1 正则的惩罚系数

当数据维度极高时可以使用，使得算法运行更快。

- lambda_bias 

在偏置上的L2正则。缺省值为0（在L1上没有偏置项的正则，因为L1时偏置不重要）

### Task Parameters

- objective [ default=reg:linear ] 
定义学习任务及相应的学习目标，可选的目标函数如下:

    - “reg:linear” –线性回归。

    - “reg:logistic” –逻辑回归。

    - “binary:logistic” –二分类的逻辑回归问题，输出为概率。

    - “binary:logitraw” –二分类的逻辑回归问题，输出的结果为wTx。

    - “count:poisson” –计数问题的poisson回归，输出结果为poisson分布。

    - 在poisson回归中，max_delta_step的缺省值为0.7。(used to safeguard optimization)

    - “multi:softmax” –让XGBoost采用softmax目标函数处理多分类问题，同时需要设置参数num_class（类别个数）

    - “multi:softprob” –和softmax一样，但是输出的是ndata * nclass的向量，可以将该向量reshape成ndata行nclass列的矩阵。每行数据表示样本所属于每个类别的概率。

    - “rank:pairwise” –set XGBoost to do ranking task by minimizing the pairwise loss

    - base_score [ default=0.5 ] 

the initial prediction score of all instances, global bias

    - eval_metric [ default according to objective ] 

校验数据所需要的评价指标，不同的目标函数将会有缺省的评价指标（rmse for regression, and error for classification, mean average precision for ranking）
    
用户可以添加多种评价指标，对于Python用户要以list传递参数对给程序，而不是map参数list参数不会覆盖’eval_metric’

The choices are listed below:

    - “rmse”: root mean square error

    - “logloss”: negative log-likelihood
    
    - “error”: Binary classification error rate. It is calculated as #(wrong cases)/#(all cases). For the predictions, the evaluation will regard the instances with prediction value larger than 0.5 as positive instances, and the others as negative instances.
    
    - “merror”: Multiclass classification error rate. It is calculated as #(wrong cases)/#(all cases).
    
    - “mlogloss”: Multiclass logloss
    
    - “auc”: Area under the curve for ranking evaluation.
    
    - “ndcg”:Normalized Discounted Cumulative Gain
    
    - “map”:Mean average precision
    
    - “ndcg@n”,”map@n”: n can be assigned as an integer to cut off the top positions in the lists for evaluation.
    
    - “ndcg-“,”map-“,”ndcg@n-“,”map@n-“: In XGBoost, NDCG and MAP will evaluate the score of a list without any positive samples as 1. By adding “-” in the evaluation metric XGBoost will evaluate these score as 0 to be consistent under some conditions. 
training repeatively

- seed [ default=0 ] 

随机数的种子。缺省值为0
可以用于产生可重复的结果（每次取一样的seed即可得到相同的随机划分）

### Console Parameters

The following parameters are only used in the console version of xgboost 
* use_buffer [ default=1 ] 

是否为输入创建二进制的缓存文件，缓存文件可以加速计算。缺省值为1 
* num_round 

boosting迭代计算次数。 
* data 

输入数据的路径 
* test:data 

测试数据的路径 
* save_period [default=0] 

表示保存第i*save_period次迭代的模型。例如save_period=10表示每隔10迭代计算XGBoost将会保存中间结果，设置为0表示每次计算的模型都要保持。 

* task [default=train] options: train, pred, eval, dump 

- train：训练模型

- pred：对测试数据进行预测 

- eval：通过eval[name]=filenam定义评价指标 

- dump：将学习模型保存成文本格式 

* model_in [default=NULL] 

- 指向模型的路径在test, eval, dump都会用到，如果在training中定义XGBoost将会接着输入模型继续训练 

* model_out [default=NULL] 

- 训练完成后模型的保存路径，如果没有定义则会输出类似0003.model这样的结果，0003是第三次训练的模型结果。 

* model_dir [default=models] 

输出模型所保存的路径。 

* fmap 

feature map, used for dump model 

* name_dump [default=dump.txt] 

name of model dump file 

* name_pred [default=pred.txt] 

预测结果文件 

* pred_margin [default=0] 

输出预测的边界，而不是转换后的概率

### sklearn参数对比
如果你比较习惯scikit-learn的参数形式，那么XGBoost的Python 版本也提供了sklearn形式的接口 XGBClassifier。它使用sklearn形式的参数命名方式，对应关系如下：

eta –> learning_rate
lambda –> reg_lambda
alpha –> reg_alpha

## 调参核心

- 调参1, 提高准确率: num_leaves, max_depth, learning_rate
- 调参2, 降低过拟合: max_bin, min_data_in_leaf
- 调参3, 降低过拟合: 正则化L1, L2
- 调参4, 降低过拟合: 数据抽样(样本抽样), 列抽样(特征抽样)
  
### 调参方向：处理过拟合（过拟合和准确率往往相反）
- 使用较小的 max_bin
- 使用较小的 num_leaves
- 使用 min_data_in_leaf 和 min_sum_hessian_in_leaf
- 通过设置 bagging_fraction 和 bagging_freq 来使用 bagging
- 通过设置 feature_fraction <1来使用特征抽样
- 使用更大的训练数据
- 使用 lambda_l1, lambda_l2 和 min_gain_to_split 来使用正则
- 尝试 max_depth 来避免生成过深的树


## XGBoost扩展
#### Xgbfi特征重要性分析
用于训练好的 xgboost 模型分析对应特征的重要性，当然你也可以使用 fmap 来观察



# XGBoost调参技巧(实战经验)
本节整理的是本人在实战过程中调惨的经验, 第一个建议就是在没有弄清每个参数对训练效果的影响时不要使用太多的参数, 刚开始就使用简单的一两个参数就好, 我在使用过程中刚开始只是指定objective和num_class即可.
## max_depth
max_depth对模型的收敛效果很明显, 刚开始我在调参过程中max_depth的值设置的比较小([0-6]之间选取), test-merror下降比较慢, 就算num_boost_round设置很大test-error也下降的很慢, 而且到一定程度就不下降了,值也比较大. 后来将max_depth设置到[20-30]之间后, train-error和test-error的值衰减的很快.所以在设置max_depth的时候不要局限在某一个范围之内.

## learning_rate
有的人为了提高训练进度, 刚开始就将learning_rate设置的比较小0.001,或者更小. 刚开始我在使用的过程也是, 但是训练的非常慢, 主要是对收敛效果没有很大的作用, 所以就暂时将这个参数去掉. 训练完成之后,将mlogloss-mean曲线和merror—mean曲线画出来之后发现一个问题, 如下图所示, 我们只看test loss 和test merror, 你会发现test loss的曲线开始上翘, 而test merror的曲线还是在减小的, 理论上从train和test loss来看模型可能出现过拟合了, 但是为什么error会一致变小呢, ***后面我尝试调整learning_rate的大小, 速率往小了调, 这时候我发现 test merror下降的速度变快, 而且可以更小***
<center><img src="/wiki/static/images/essemble/xgboost/xgboost_1.png" alt="xgboost-1"/></center>

在我调整learning_rate之后, 训练结果如下: 图像上可以看出,设定learning_rate起到了效果, 而且设定在其他参数不变的情况的, 模型并没有过拟合.
<center><img src="/wiki/static/images/essemble/xgboost/xgboost_2.png" alt="xgboost-2"/></center>





# 参考文献
[贝叶斯优化调参实战（随机森林，lgbm波士顿房价）](https://blog.csdn.net/ssswill/article/details/86564056)


[Bayesian Optimization of XGBoost Parameters](https://www.kaggle.com/tilii7/bayesian-optimization-of-xgboost-parameters/notebook)

[BayesBoost](https://github.com/mpearmain/BayesBoost)

[贝叶斯优化在XGBoost及随机森林中的使用](https://yq.aliyun.com/articles/709047)

[Hyperparameter tuning](https://www.kaggle.com/fabiendaniel/hyperparameter-tuning/notebook)

[sklearn、XGBoost、LightGBM的文档阅读小记（转载）](https://www.jianshu.com/p/dcaff7a1e7d8)

[xgboost使用技巧](https://www.lizenghai.com/archives/29288.html)

[XGBoost-Python完全调参指南-介绍篇](https://blog.csdn.net/wzmsltw/article/details/50988430)

[XGBoost-Python完全调参指南-参数解释篇](https://blog.csdn.net/wzmsltw/article/details/50994481)

[XGboost数据比赛实战之调参篇(完整流程)](https://segmentfault.com/a/1190000014040317)

[XGBoost python调参示例](https://blog.csdn.net/weiyongle1996/article/details/78360873)

[XGBoost参数调优完全指南（附Python代码）](https://blog.csdn.net/u010657489/article/details/51952785)

[xgboost原理及应用--转](https://www.cnblogs.com/zhouxiaohui888/p/6008368.html)

[xgboost 调参经验](https://blog.csdn.net/u010414589/article/details/51153310/)

[xgboost使用调参](https://blog.csdn.net/q383700092/article/details/53763328)

[XGBoost 与 Boosted Tree](http://www.52cs.org/?p=429)

[机器学习系列(11)_Python中Gradient Boosting Machine(GBM）调参方法详解](https://blog.csdn.net/han_xiaoyang/article/details/52663170)

[机器学习时代的三大神器](http://www.360doc.com/content/18/0101/17/40769523_718161675.shtml)

[xgboost原理及应用--转](https://www.cnblogs.com/zhouxiaohui888/p/6008368.html)

[GBDT、XGBoost、LightGBM 的使用及参数调优](https://www.jianshu.com/p/0fe45d4e9542)

[机器学习---xgboost与lightgbm效果比较（2）](https://blog.csdn.net/zhouwenyuan1015/article/details/77481184)

[XGBoost调参demo（Python）](https://blog.csdn.net/hx2017/article/details/78064362)

[lightgbm,xgboost,gbdt的区别与联系](https://www.cnblogs.com/mata123/p/7440774.html)

[Python超参数自动搜索模块GridSearchCV上手](https://www.cnblogs.com/nwpuxuezha/p/6618205.html)
[Sklearn中的CV与KFold详解](https://blog.csdn.net/FontThrone/article/details/79220127)

[史上最详细的XGBoost实战](https://blog.csdn.net/u013709270/article/details/78156207)

[XGBoost特征重要性以及CV](https://www.jianshu.com/p/0a8a5f80161e)

[XGBoost 核心数据结构和API(速查表)](https://blog.csdn.net/maerdym/article/details/84863113)

[XGBoost和LightGBM的参数以及调参](https://www.jianshu.com/p/1100e333fcab)

[实战微博互动预测之三_xgboost答疑解惑](http://www.voidcn.com/article/p-zxyduztr-brm.html)