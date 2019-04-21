---
layout: post
title: scikit-learn之模型存储与上线
date: 2019-04-21
tags: python
---  
### 模型存储
- pickle保存模型到本地，不能跨平台使用
  ```python
  >>> from sklearn import svm
  >>> from sklearn import datasets
  >>> clf = svm.SVC()
  >>> iris = datasets.load_iris()
  >>> X, y = iris.data, iris.target
  >>> clf.fit(X, y)  
  SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
      decision_function_shape=None, degree=3, gamma='auto', kernel='rbf',
      max_iter=-1, probability=False, random_state=None, shrinking=True,
      tol=0.001, verbose=False)

  >>> import pickle
  >>> with open('path/to/clf.pkl','wb') as file:
  >>>   pickle.dump(clf, file)
  >>> with open('path/to/clf.pkl','rb') as file:
  >>>   clf2 = pickle.load(file)
  >>> clf2.predict(X[0:1])
  array([0])
  >>> y[0]
  0
  ```
  另一种写法
  ```python
  >>> import pickle
  >>> s = pickle.dumps(clf)
  >>> f = open('path/to/clf.pkl','w')
  >>> f.write(s)
  >>> f.close()
  >>> f2 = open('path/to/clf.pkl','r')
  >>> s2 = f2.read()
  >>> clf2 = pickle.loads(s2)
  >>> clf2.predit(X[0:1])
  ```
- joblib保存模型到本地，不能跨平台使用
  ```python
  >>> from sklearn.externals import joblib
  >>> joblib.dump(clf, 'path/to/clf.model')
  >>> clf = joblib.load('path/to/clf.model')
  ```
- PMML跨平台使用，实现模型上线
  跨平台使用时可以根据`sklearn2pmml`转成pmml文件，通过jpmml去部署到线上。例如模型上线，很可能是python训练模型后，然后使用java正式上线运行，这时可以使用pmml，通过[sklearn2pmml](https://github.com/jpmml/sklearn2pmml)模块将模型转换为PMML。   
  可参考：[Java PMML API](https://github.com/jpmml/)  
  安装sklearn2pmml模块  
  ```shell
  pip install --user --upgrade git+https://github.com/jpmml/sklearn2pmml.git
  ```
  简单示例  
  ```python
  import pandas

  iris_df = pandas.read_csv("Iris.csv")

  from sklearn.tree import DecisionTreeClassifier
  from sklearn2pmml.pipeline import PMMLPipeline

  pipeline = PMMLPipeline([
  	("classifier", DecisionTreeClassifier())
  ])
  pipeline.fit(iris_df[iris_df.columns.difference(["Species"])], iris_df["Species"])

  from sklearn2pmml import sklearn2pmml

  sklearn2pmml(pipeline, "DecisionTreeIris.pmml", with_repr = True)
  ```
  复杂示例  
  ```python
  import pandas

  iris_df = pandas.read_csv("Iris.csv")

  from sklearn_pandas import DataFrameMapper
  from sklearn.decomposition import PCA
  from sklearn.feature_selection import SelectKBest
  from sklearn.preprocessing import Imputer
  from sklearn.linear_model import LogisticRegression
  from sklearn2pmml.decoration import ContinuousDomain
  from sklearn2pmml.pipeline import PMMLPipeline

  pipeline = PMMLPipeline([
  	("mapper", DataFrameMapper([
  		(["Sepal.Length", "Sepal.Width", "Petal.Length", "Petal.Width"], [ContinuousDomain(), Imputer()])
  	])),
  	("pca", PCA(n_components = 3)),
  	("selector", SelectKBest(k = 2)),
  	("classifier", LogisticRegression())
  ])
  pipeline.fit(iris_df, iris_df["Species"])

  from sklearn2pmml import sklearn2pmml

  sklearn2pmml(pipeline, "LogisticRegressionIris.pmml", with_repr = True)
  ```   

  **实例**：[用PMML实现机器学习模型的跨平台上线](https://www.cnblogs.com/pinard/p/9220199.html)
  [完整代码](https://github.com/ljpzzz/machinelearning/tree/master/model-in-product/sklearn-jpmml)  

### pickle vs pmml vs ...
pickle和joblib的不好之处在于它们没办法兼容所有版本的sklearn，如果sklearn升级，可能会引起模型出错，所以建议使用同一个版本的sklearn。[参考链接](https://codeday.me/bug/20190202/586839.html)
