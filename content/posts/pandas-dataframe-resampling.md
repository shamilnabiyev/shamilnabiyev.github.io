---
author: "Shamil Nabiyev"
title: "Pandas Dataframe resampling"
date: "2022-07-17"
tags: [
    "tag1",
    "tag2"
]
draft: false
---

Resampling `pandas` dataframe to calculate mean/median for a selected window size.


```python
import pandas as pd

millisecond = "ms"
idx = pd.date_range('1/1/2022', periods=100, freq=millisecond)
series = pd.Series(list(range(100, 200)), index=idx)
df = pd.DataFrame({'s': series})

window_size = 10
df.resample(f"{window_size}{millisecond}").mean()
df.resample(f"{window_size}{millisecond}").median()
df.resample(f"{window_size}{millisecond}").last()
```