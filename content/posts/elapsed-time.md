---
author: "Shamil Nabiyev"
title: "Elapsed time"
date: "2022-07-25"
tags: [
    "python",
    "time"
]
draft: false
---

Python script to measure the elapsed time in seconds

```python
from timeit import default_timer as timer
from datetime import timedelta

start = timer()

# Simulate a long lasting calculation/process
for i in range(0, 99999):
    k = 0
    for m in range(0, 999):
        k = (i - 1) + 10
# End of simulation    

end = timer()
td = timedelta(seconds=end-start)

print("Elapsed time:", td)
# Output
# Elapsed time: 00:01:15.375

print("Elapsed time:", td.seconds, "seconds")
# Output
# Elapsed time: 75 seconds
```