---
layout: post
title: 机器学习之pipeline
date: 2019-04-22
tags: 机器学习
---  
### pipeline简介
一个典型的机器学习过程通常会包含：源数据ETL，数据预处理，特征提取，模型训练与交叉验证，新数据预测等。我们可以将这个包含多个步骤的流水线式工作结合在一起构成一条管道，称为pipeline。    
数据沿pipeline“流动”，从原始格式，最终得到有用的信息。pipeline中每一步的输入的数据，都是经过前一步处理过的，也就是某一数据处理单元的输出，是下一步的输入。pipeline中所有步骤除了最后一步（estimators），都要有fit和transform方法，数据呈现如下流动形式（即input1 -> output1(input2) -> output2(input3) -> ...）。最后一步(estimators)只有fit方法被使用，最后做模型的检验评估。        

**重要概念**
- transformer
  转换器。数据预处理过程，包括特征标准化、均一化、PCA降维、LDA降维等。
- estimator   
  评估器或适配器。通常是使用的算法，如线性回归、逻辑回归、SVM等。
- pipeline对象
  pipeline对象接受二元tuple构成的list，每一个二元tuple中的第一个元素为任意指定字符串(arbitrary identifier string)，用以获取pipeline object中的单独元素(individual elements)，二元tuple中的第二个元素是scikit-learn与之相适配的transformer或者estimator。    

**示例**
```python
from sklearn.preprocessing import StandardScalerfrom sklearn.decomposition import PCAfrom sklearn.linear_model import LogisticRegressionfrom sklearn.pipeline import Pipeline

pipe_lr = Pipeline([('sc', StandardScaler()),
                    ('pca', PCA(n_components=2)),
                    ('clf', LogisticRegression(random_state=1))
                    ])
pipe_lr.fit(X_train, y_train)
print('Test accuracy: %.3f' % pipe_lr.score(X_test, y_test))
```   
`StandardScaler`和`PCA`构成中间步骤的`transformer`，`LogisticRegression`作为最终的`estimator`。    
当我们执行`pipe_lr.fit(X_train, y_train)`时，首先由`StandardScaler`在训练集上执行 `fit`和`transform`方法，`transformed`后的数据又被传递给Pipeline对象的下一步，也即`PCA()`。和StandardScaler一样，PCA也是执行`fit`和`transform`方法，最终将转换后的数据传递给`LosigsticRegression`。     

### scikit-learn Pipline  

- **实例**：[搭建一个简单的机器学习流水线](https://zhuanlan.zhihu.com/p/37650986)    

- pipline合并
  pipeline可以只包含数据处理的方法，即没有应用最后的算法，这样叫做Transformation Pipeline，这种pipeline可以平行处理numerical feature和categorical feature，最后将两者结果合并。
  ```python
  from sklearn.pipeline import FeatureUnion
  from sklearn.base import BaseEstimator, TransformerMixin
  from sklearn.datasets import fetch_california_housing

  housing = fetch_california_housing()
  housing_num = housing.drop("ocean_proximity", axis=1)

  num_attribs = list(housing_num) # housing_num是只包含numerical data的dataframe，加上list方法是把所有attributes放入一个列表
  cat_attribs = ["ocean_proximity"]

  num_pipeline = Pipeline([
                         ('selector', DataFrameSelector(num_attribs)),
                         ('imputer', Imputer(strategy="median")),
                         ('std_scaler', StandardScaler()),
  ])

  cat_pipeline = Pipeline([
                         ('selector', DataFrameSelector(cat_attribs)),
                         ('label_binarizer', LabelBinarizer()),
  ])

  full_pipeline = FeatureUnion(transformer_list=[
                                                ("num_pipeline", num_pipeline),
                                                ("cat_pipeline", cat_pipeline),
  ])
  ```   
  最后把transformation pipeline和最后的预测模型连接成一个总的pipeline。
  ```python
  full_pipeline_with_predictor = Pipeline([
          ("preparation", full_pipeline),
          ("linear", LinearRegression())
      ])
  full_pipeline_with_predictor.fit(housing, housing_labels)
  full_pipeline_with_predictor.predict(test_data)
  ```

### Spark ML Pipeline
[官方文档](http://spark.apache.org/docs/latest/ml-guide.html)    
Spark ML Pipeline基于DataFrame构建了一套High-level API，我们可以使用MLPipeline构建机器学习应用，它能够将一个机器学习应用的多个处理过程组织起来，通过在代码实现的级别管理好每一个处理步骤之间的先后运行关系，极大地简化了开发机器学习应用的难度。    

**重要概念**
- DataFrame
  Spark ML Pipeline使用DataFrame作为机器学习输入输出数据集的抽象。DataFrame可以基于不同的数据源进行构建，比如结构化文件、Hive表、数据库、RDD等。    
- Transformer
  Transformer对机器学习中要处理的数据集进行转换操作，类似于Spark中对RDD进行Transformation操作（对一个输入RDD转换处理后生成一个新的RDD）。    
  Transformer类继承关系:![](https://jacky-wangjj.github.io/images/blog/machine-learning/spark-ml-pipeline-transformers.png#pic_center)     
  Transformer定义如下：
  ```scala
  package org.apache.spark.ml

  @DeveloperApi
  abstract class Transformer extends PipelineStage {

    @Since("2.0.0")
    @varargs
    def transform(
        dataset: Dataset[_],
        firstParamPair: ParamPair[_],
        otherParamPairs: ParamPair[_]*): DataFrame = {
      val map = new ParamMap()
        .put(firstParamPair)
        .put(otherParamPairs: _*)
      transform(dataset, map)
    }

    @Since("2.0.0")
    def transform(dataset: Dataset[_], paramMap: ParamMap): DataFrame = {
      this.copy(paramMap).transform(dataset)
    }

    @Since("2.0.0")
    def transform(dataset: Dataset[_]): DataFrame

    override def copy(extra: ParamMap): Transformer
  }
  ```  

- Estimator
  Estimator用来训练模型，它的输入是一个DataFrame，经过训练最终输出一个Model，Model是Spark ML中对机器学习模型的抽象和定义，Estimator类使用`fit`方法来实现对模型的训练。    
  代码如下：    
  ```scala
  package org.apache.spark.ml

  @DeveloperApi
  abstract class Estimator[M <: Model[M]] extends PipelineStage {

    @Since("2.0.0")
    @varargs
    def fit(dataset: Dataset[_], firstParamPair: ParamPair[_], otherParamPairs: ParamPair[_]*): M = {
      val map = new ParamMap()
        .put(firstParamPair)
        .put(otherParamPairs: _*)
      fit(dataset, map)
    }

    @Since("2.0.0")
    def fit(dataset: Dataset[_], paramMap: ParamMap): M = {
      copy(paramMap).fit(dataset)
    }

    @Since("2.0.0")
    def fit(dataset: Dataset[_]): M

    @Since("2.0.0")
    def fit(dataset: Dataset[_], paramMaps: Array[ParamMap]): Seq[M] = {
      paramMaps.map(fit(dataset, _))
    }

    override def copy(extra: ParamMap): Estimator[M]
  }
  ```  

- Parameter  
  Transformer和Estimator中的参数，是一套公用的API。

- PipelineStage  
  PipelineStage是构建一个Pipeline的基本元素，它或者是一个Transformer，或者是一个Estimator。    

- Pipeline  
  一个Pipeline是基于多个PipelineStage构建而成的DAG图，简单一点可以使用线性的PipelineStage序列来完成机器学习应用的构建，当然也可以构建相对复杂的PipelineStage DAG图。   
  调用Pipeline的fit方法，会生成一个PipelineModel，PipelineModel在训练阶段和测试阶段之前比较重要的一个PipelineStage，它起了承上启下的作用，调用PipelineModel的transform方法，按照和训练阶段类似的数据处理（转换）流程，经过相同的各个PipelineStage对数据集进行变换，最后将训练阶段生成的模型作用在测试数据集上，从而实现最终的预测目的。    

[参考链接](http://shiyanjun.cn/archives/1693.html)    
