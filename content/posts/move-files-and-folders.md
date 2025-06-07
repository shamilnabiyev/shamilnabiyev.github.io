---
title: "Move files and folders"
date: "2019-06-19"
description: "Move files and folders"
tags: [
    "python"
]
draft: false
---

Move files and folder to another destination using python.

```python
import os

basedir = 'files'
new_dst = "new_dst_folder/"

for folder in os.listdir(basedir):
    inner_path = basedir + folder + "/"
    for file in os.listdir(inner_path):
      shutil.move(os.path.join(inner_path, file), new_dst)
```
