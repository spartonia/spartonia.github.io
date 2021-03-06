---
layout: post
title: Chooosing Best Predictive Model with Regression
categories: [Machine Learning]
tags: [regression, sklearn]
comments: true
---

### Introduction 
The aim of this projcet is to  build  a predictive model for the given data set. Two datasets are provided; train and test data sets. There are 254 features  in for each set and a target value for training set. Ultimately, the goal is to assess predictive accuracy on the test set using the mean squared error metric. 

Before starting to inspect the data we need to import required tools. We will use `pandas` to inspect the data, and `numpy` and `sklearn` for implementing predictor models. 


```python
import os 
import time

import pandas as pd 
import numpy as np 
import cPickle
```


```python
from sklearn.preprocessing import scale
from sklearn.decomposition import PCA
from sklearn.cross_validation import KFold
from sklearn.linear_model import LinearRegression, ElasticNetCV, RidgeCV, LassoCV 
from sklearn.preprocessing import LabelEncoder
from sklearn.ensemble import RandomForestRegressor
from sklearn.svm import LinearSVR, NuSVR, SVR
```

    /home/spartonia/.venv/experiments/lib/python2.7/site-packages/sklearn/cross_validation.py:44: DeprecationWarning: This module was deprecated in version 0.18 in favor of the model_selection module into which all the refactored classes and functions are moved. Also note that the interface of the new CV iterators are different from that of this module. This module will be removed in 0.20.
      "This module will be removed in 0.20.", DeprecationWarning)


We also add the following to suppress warnings. 


```python
import warnings
warnings.filterwarnings('ignore') 
```

Data Inspection
---------------
We load train and test data sets and separate `X` and `y` of training set accordingly. 


```python
data_dir = '.'
df = pd.read_csv(os.path.join(data_dir, 'codetest_train.txt'), delimiter='\t')
X_test = pd.read_csv(os.path.join(data_dir, 'codetest_test.txt'), delimiter='\t')
```

To start off, we print the head of dataset 


```python
df.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>target</th>
      <th>f_0</th>
      <th>f_1</th>
      <th>f_2</th>
      <th>f_3</th>
      <th>f_4</th>
      <th>f_5</th>
      <th>f_6</th>
      <th>f_7</th>
      <th>f_8</th>
      <th>...</th>
      <th>f_244</th>
      <th>f_245</th>
      <th>f_246</th>
      <th>f_247</th>
      <th>f_248</th>
      <th>f_249</th>
      <th>f_250</th>
      <th>f_251</th>
      <th>f_252</th>
      <th>f_253</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3.066056</td>
      <td>-0.653</td>
      <td>0.255</td>
      <td>-0.615</td>
      <td>-1.833</td>
      <td>-0.736</td>
      <td>NaN</td>
      <td>1.115</td>
      <td>-0.171</td>
      <td>-0.351</td>
      <td>...</td>
      <td>-1.607</td>
      <td>-1.400</td>
      <td>-0.920</td>
      <td>-0.198</td>
      <td>-0.945</td>
      <td>-0.573</td>
      <td>0.170</td>
      <td>-0.418</td>
      <td>-1.244</td>
      <td>-0.503</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-1.910473</td>
      <td>1.179</td>
      <td>-0.093</td>
      <td>-0.556</td>
      <td>0.811</td>
      <td>-0.468</td>
      <td>-0.005</td>
      <td>-0.116</td>
      <td>-1.243</td>
      <td>1.985</td>
      <td>...</td>
      <td>1.282</td>
      <td>0.032</td>
      <td>-0.061</td>
      <td>NaN</td>
      <td>-0.061</td>
      <td>-0.302</td>
      <td>1.281</td>
      <td>-0.850</td>
      <td>0.821</td>
      <td>-0.260</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.830711</td>
      <td>0.181</td>
      <td>-0.778</td>
      <td>-0.919</td>
      <td>0.113</td>
      <td>0.887</td>
      <td>-0.762</td>
      <td>1.872</td>
      <td>-1.709</td>
      <td>0.135</td>
      <td>...</td>
      <td>-0.237</td>
      <td>-0.660</td>
      <td>1.073</td>
      <td>-0.193</td>
      <td>0.570</td>
      <td>-0.267</td>
      <td>1.435</td>
      <td>1.332</td>
      <td>-1.147</td>
      <td>2.580</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 255 columns</p>
</div>



It seems the data is mostly in numeric format, including some missing values as well. The target is first column and the rest of columns represent fearures. So we can separate X and y


```python
X = df.iloc[:, 1:]
y = df.target
y = y.values  # To np.array
```

Now we need to make sure that all columns of X are in numeric format.


```python
print set(X.dtypes)
```

    set([dtype('O'), dtype('float64')])


Seems that we have `dtype('O')` along with `float64` too. So we inspect more those columns separately.


```python
categorical_cols = []
for col in X.columns:
    if X[col].dtype != 'float64':
        categorical_cols.append(col)
        print 'column:', col 
        print X[col].head(3)
        print '-' * 15
```

    column: f_61
    0    b
    1    a
    2    b
    Name: f_61, dtype: object
    ---------------
    column: f_121
    0    D
    1    A
    2    B
    Name: f_121, dtype: object
    ---------------
    column: f_215
    0       red
    1      blue
    2    orange
    Name: f_215, dtype: object
    ---------------
    column: f_237
    0    Canada
    1    Canada
    2    Canada
    Name: f_237, dtype: object
    ---------------


As we can see, we have a few columns with non-numeric data types (categorical), so we should convert them to numeric ones. Columns to be decoded:  [f_61, f_121, f_215, f_237] should be decoded.

We add dummy columns to encode categorical data. 


```python
X[categorical_cols].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>f_61</th>
      <th>f_121</th>
      <th>f_215</th>
      <th>f_237</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>b</td>
      <td>D</td>
      <td>red</td>
      <td>Canada</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a</td>
      <td>A</td>
      <td>blue</td>
      <td>Canada</td>
    </tr>
    <tr>
      <th>2</th>
      <td>b</td>
      <td>B</td>
      <td>orange</td>
      <td>Canada</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a</td>
      <td>C</td>
      <td>blue</td>
      <td>USA</td>
    </tr>
    <tr>
      <th>4</th>
      <td>b</td>
      <td>E</td>
      <td>orange</td>
      <td>Canada</td>
    </tr>
  </tbody>
</table>
</div>



As we saw earlier the data inclued missing values too. So we need to fix those raws with null values. We could either remove all raws with missing values but it seems like a good idea, beacuse we will miss majority of data:


```python
print X.dropna().shape
```

    (32, 254)


So only have 32 samples without missing values. Another approach is to use interploation to fill missing values. We use interpolation with `limit=2` and in both directions to fill null values. We also scale the data such that all values fall in the same interval. This will speed up the training process. The following function performs these 3 operations on the data.


```python
def preprocess_data(data, categorical_cols):
    """Preprocess and clean the data:
        * add dummy clumns for categorical values
        * fix missing values
        * scale 
    """
    # dd dummy clumns for categorical values
    X_categorical = pd.get_dummies(data[categorical_cols])
    X = pd.concat([data.drop(categorical_cols, axis=1), X_categorical], axis=1)
    
    # Use interpolation to deal with missing values. 
    X_filled = X.interpolate(limit=2,limit_direction='both').dropna()
    
    # Scale
    X_scaled = pd.DataFrame(data=scale(X_filled), columns=X.columns)
    
    return X_scaled
```


```python
X_scaled = preprocess_data(X, categorical_cols)
```

The next thing we can try is to use PCA to see if we can reduce the dimension (number of features):


```python
# Do some PCA to see if we can reduce the dimension..
pca = PCA()
```


```python
# Scale and transform data to get principal components
X_reduced = pca.fit_transform(X_scaled)
```


```python
# Variance (% cumulative) explained by the principal components
print np.cumsum(np.round(pca.explained_variance_ratio_, decimals=4) * 100)[::-1]
```

    [ 99.96  99.95  99.94  99.93  99.92  99.82  99.72  99.61  99.5   99.39
      99.16  98.93  98.7   98.46  98.22  97.98  97.74  97.5   97.26  97.02
      96.77  96.52  96.27  96.02  95.77  95.52  95.27  95.01  94.75  94.49
      94.23  93.97  93.71  93.45  93.18  92.91  92.64  92.37  92.1   91.83
      91.56  91.29  91.01  90.73  90.45  90.17  89.89  89.61  89.33  89.05
      88.77  88.49  88.21  87.92  87.63  87.34  87.05  86.76  86.47  86.18
      85.89  85.6   85.31  85.01  84.71  84.41  84.11  83.81  83.51  83.21
      82.91  82.61  82.3   81.99  81.68  81.37  81.06  80.75  80.44  80.13
      79.82  79.51  79.2   78.88  78.56  78.24  77.92  77.6   77.28  76.96
      76.64  76.32  76.    75.68  75.35  75.02  74.69  74.36  74.03  73.7
      73.37  73.04  72.71  72.37  72.03  71.69  71.35  71.01  70.67  70.33
      69.99  69.65  69.31  68.96  68.61  68.26  67.91  67.56  67.21  66.86
      66.51  66.16  65.81  65.45  65.09  64.73  64.37  64.01  63.65  63.29
      62.93  62.57  62.21  61.84  61.47  61.1   60.73  60.36  59.99  59.62
      59.25  58.88  58.51  58.13  57.75  57.37  56.99  56.61  56.23  55.85
      55.47  55.08  54.69  54.3   53.91  53.52  53.13  52.74  52.35  51.96
      51.57  51.17  50.77  50.37  49.97  49.57  49.17  48.77  48.37  47.97
      47.56  47.15  46.74  46.33  45.92  45.51  45.1   44.69  44.28  43.86
      43.44  43.02  42.6   42.18  41.76  41.34  40.91  40.48  40.05  39.62
      39.19  38.76  38.33  37.9   37.46  37.02  36.58  36.14  35.7   35.26
      34.82  34.38  33.93  33.48  33.03  32.58  32.13  31.68  31.23  30.78
      30.32  29.86  29.4   28.94  28.48  28.01  27.54  27.07  26.6   26.13
      25.66  25.19  24.71  24.23  23.75  23.27  22.79  22.31  21.82  21.33
      20.84  20.35  19.86  19.37  18.88  18.38  17.88  17.38  16.88  16.37
      15.86  15.35  14.84  14.33  13.82  13.3   12.78  12.26  11.74  11.21
      10.68  10.15   9.61   9.07   8.53   7.98   7.43   6.88   6.32   5.76
       5.19   4.62   4.01   3.39   2.74   2.08   1.4    0.7 ]


As we can see from above, seems that all components are required to explain the variance in data. So we'll use all components.

Once the data is cleaned, we are ready to test a few regressors. We create a dictionary called 'models', which includes name to regressor items. By this, we can train all models together and compatre the results to chhose the best performing model. Feel free to uncomment any of models to include in the results. 


```python


models = {
    'LinearRegression': LinearRegression(fit_intercept=True),
    'ElasticNetCV': ElasticNetCV(fit_intercept=True,n_jobs=-1),
    'LassoCV': LassoCV(n_jobs=-1),
    'LinearSVR': LinearSVR(), 
    'Ridge': RidgeCV(),
#     'NuSVR': NuSVR(),
#     'SVR': SVR(),
#     'RFRegressor': RandomForestRegressor(n_estimators=50, n_jobs=-1)  # --> Dangerous! Very time consuming..
}
```

We also use cross validation to choose the best model. 


```python
n_folds = 20
kf = KFold(X_reduced.shape[0], n_folds=n_folds)
err = dict.fromkeys(models.keys(), 0)
```

for each model, we run a k-fold cross validation and save the results in a dict, one result for each model. 


```python
for model_name, model in models.iteritems():
    print 'training', model_name, '..'
    t0 = time.time()
    for train, test in kf: 
        model.fit(X_scaled.values[train], y[train])
    
        p = np.squeeze(map(model.predict, X_scaled.values[test]))
        e = p - y[test]
        error = np.dot(e,e)
        err[model_name] += error

    print '\t..', time.time()-t0, 'seconds.'
print 'Done!'
```

    training Ridge ..
    	.. 8.10740995407 seconds.
    training LinearSVR ..
    	.. 26.9117820263 seconds.
    training LinearRegression ..
    	.. 3.52279186249 seconds.
    training ElasticNetCV ..
    	.. 9.98840403557 seconds.
    training LassoCV ..
    	.. 10.8082270622 seconds.
    Done!


Once training is done, we can calculate the `rmse` for each model. The results for 10, and 20 fold cross validations are listed below.  


```python
print 'RMSE on {}-fold CV:'.format(n_folds)
for model, error in err.iteritems():
    rmse_10cv = np.sqrt(error/X_scaled.shape[0])
    print model, rmse_10cv    
```

    RMSE on 20-fold CV:
    LinearRegression 3.50829586837
    LinearSVR 3.47428941377
    Ridge 3.50748745066
    ElasticNetCV 3.43133680584
    LassoCV 3.41826205147


#### RMSE on 20-fold CV:
* Ridge 3.50748745066
* ElasticNetCV 3.49292650999
* LinearSVR 3.47267712099
* LinearRegression 3.50829586837
* LassoCV 3.4933509662
* SVR 3.76794728891

The best performing model is `RFRegressor` followed by `LassoCV`, both trained on 20-fold cross validation. Considering the training time, which is huge fir `RFRegressor`, we choose second best performing i.e. `LassoCV`. Now we traing a new lassoCV model with all data to use for predicting test data. 


```python
regressor = LassoCV(cv=20)
regressor.fit(X_scaled, y)
```




    LassoCV(alphas=None, copy_X=True, cv=20, eps=0.001, fit_intercept=True,
        max_iter=1000, n_alphas=100, n_jobs=1, normalize=False, positive=False,
        precompute='auto', random_state=None, selection='cyclic', tol=0.0001,
        verbose=False)




```python
regressor.coef_[0:6]
```




    array([ 0., -0., -0.,  0.,  0.,  0.])



Before using the predictor on test data, we need to make sure that test data has the same format as train. So we call the preprocessing function again, with same settings, to unify data formats. 


```python
X_test_scaled = preprocess_data(X_test, categorical_cols)
```

We can now predict the test data and write the results to a txt file. 


```python
y_test = regressor.predict(X_test_scaled)
```


```python
result_file = os.path.join(data_dir, 'prediction.txt')
np.savetxt(result_file, y_test, delimiter='\n', fmt='%1.6f')
```

We can also get to n features and therir weights: 


```python
coef_idx = [i for i in range(len(regressor.coef_))]
lst = zip(abs(regressor.coef_), coef_idx)
# sort the list 
lst_sort = sorted(lst, reverse=True)
```


```python
top_n_feats = 10
print 'Top {} fetures'.format(top_n_feats)
print '=' * 20
for i in lst_sort[:10]:
    print X_scaled.columns[i[1]] , '-->', i[0]
```

    Top 10 fetures
    ====================
    f_175 --> 2.77409249023
    f_205 --> 1.70538319659
    f_61_e --> 1.24219269182
    f_35 --> 0.808882249231
    f_218 --> 0.759748432363
    f_61_c --> 0.577622944907
    f_237_Mexico --> 0.41012307336
    f_94 --> 0.33767073516
    f_237_Canada --> 0.309637288423
    f_195 --> 0.12192988248



```python
# Top 10 fetures
# ====================
# f_175 --> 2.77409249023
# f_205 --> 1.70538319659
# f_61_e --> 1.24219269182
# f_35 --> 0.808882249231
# f_218 --> 0.759748432363
# f_61_c --> 0.577622944907
# f_237_Mexico --> 0.41012307336
# f_94 --> 0.33767073516
# f_237_Canada --> 0.309637288423
# f_195 --> 0.12192988248
```

Save the model into disk:


```python
save_model_name = 'Best_model.pkl'
with open(os.path.join(data_dir, save_model_name), 'wb') as f: 
    cPickle.dump(regressor, f)
```


```python

```
