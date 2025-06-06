---
author: "Shamil Nabiyev"
title: "Pandas Multi-Index"
date: "2022-07-14"
tags: [
    "python",
    "pandas"
]
draft: false
---

Pandas DataFrame create multiindex using existing columns 

```python
import pandas as pd

df = pd.read_csv("./data/dataset.csv")

df = df.set_index(["INSTANCES", "TIMEPOINTS"], inplace=False)
```