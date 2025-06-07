---
title: "Numpy3d array to Sktime panel data"
date: "2022-05-22"
tags: [
    "python",
    "timeseries",
    "numpy",
    "sktime"
]
draft: false
---

Convert a `numpy3d` data to nested Panel data to use with `sktime` library 

```python
import numpy as np
from sktime.datatypes import convert
from sktime.datatypes._panel._convert import from_multi_index_to_nested

# import the numpy3d data
# X_train has the shape of (num_instances, num_features, num_timepoints)
X_train = np.load('./data/X_train.npy')

# Convert the numpy3d data to Multi-Index Pandas DataFrame
X_train = convert(X_train, from_type="numpy3D", to_type="pd-multiindex")

# Convert the Multi-Index Pandas DataFrame to Panel Data
X_train = from_multi_index_to_nested(X_train)

# Shorter solution to convert a numpy3d to the panel data format
X_train = from_3d_numpy_to_nested(
  np.load('./data/X_train.npy')
)
```




Numpy3D Array:

<center>
    <img src="/img/numpy3d-array.png" alt="numpy3d-array" width="400"/>
</center>

Multi-Index Pandas DataFrame:

<center>
    <img src="/img/pandas-df-multiindex.png" alt="pandas-df-multiindex" width="300"/>
</center>

Sktime panel data:

<center>
    <img src="/img/sktime-panel-data.png" alt="sktime-panel-data" width="600"/>
</center>
