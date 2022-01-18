# Dense Feature Mixer

## Introduction

Dense Feature Mixer (DFM) is a Scikit-Learn-compliant transformer that converts categorical variables to numerical vector representations. This is achieved by creating a small multilayer perceptron architecture in which each categorical variable is passed through an embedding layer, for which weights are extracted and turned into DataFrame columns.

## Usage

DFM works like any Scikit-Learn transformer, the only difference being that it requires `y` to be passed as it is the neural network's target.

The `numeric_vars` argument is optional. If available, these variables will be included as an additional input to the neural network. This may reduce the interpretability of the final model as the complete effect of numeric variables on the target variable can become unclear.

```python
from dense_feature_mixer import DenseFeatureMixer

dfm = DenseFeatureMixer(task="regression", categorical_vars=["..."], numeric_vars=["..."])
dfm.fit(X=X_train, y=y_train)
output = dfm.transform(X=X_train)
```

By default DFM will convert categorical variables into integer arrays by applying scikit's `OrdinalEncoder`.

DFM can be included in pipelines. However, if `categorical_vars` is specificed, DFM has to be the first step in the pipeline, or previous transformations have to be held inside the provided `ColumnTransformerWithNames`. This is because DFM requires its `X` input to be a `DataFrame` with proper column names, which cannot be guaranteed if previous transformations are applied as is.

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.compose import ColumnTransformer

from dense_feature_mixer import DenseFeatureMixer

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

dfm = DenseFeatureMixer(task="classification", categorical_vars=categorical_vars,
                        numeric_vars=numeric_vars, encode=True, unknown_category=999)
num_pipe = make_pipeline(SimpleImputer(strategy="mean"), StandardScaler())
cat_transformer = SimpleImputer(strategy="most_frequent")
col_transformer = ColumnTransformerWithNames([("num_transformer", num_pipe, numeric_vars),
                                              ("cat_transformer", cat_transformer, categorical_vars)])

pipe = make_pipeline(col_transformer,
                     dfm,
                     LogisticRegression())
pipe.fit(X_train, y_train)
```