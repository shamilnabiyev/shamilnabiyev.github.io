---
title: "Sklearn data imputation"
date: "2021-11-09"
tags: [
    "python",
    "sklearn"
]
draft: false
---

Impute categorial and numeric features using `sklearn.impute.SimpleImputer`

```python
from sklearn.impute import SimpleImputer
import pandas as pd

df = pd.read_csv('path/to/dataset.csv')

# select categorial and numeric columns
categorial_columns = df.select_dtypes(include='object').columns
num_columns = df.columns.difference(categorial_columns)

# replace missing values using the 'most_frequent' strategy. 
most_frequent_imputer = SimpleImputer(strategy='most_frequent')
df[categorial_columns] = most_frequent_imputer.fit_transform(df[categorial_columns])

# replace missing values using the 'median' strategy. 
median_imputer = SimpleImputer(strategy='median')
df[num_columns] = median_imputer.fit_transform(df[num_columns])
```
