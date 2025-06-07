---
title: "Rename using Python"
date: "2019-06-02"
description: " Rename multiple subfolders and files in python"
tags: [
    "python"
]
draft: false
---

The following code snippet renames multiple subfolders and files.

```python
import os

basedir = "./"
for folder in os.listdir(basedir):
  new_folder_name = folder.translate(str.maketrans({" ":  r"-",".":  r"_"})) + "_"
  os.rename(os.path.join(basedir, folder), os.path.join(basedir, new_folder_name) )
  
  
for folder in os.listdir(basedir):
  inner_path = basedir + folder + "/"
  for file in os.listdir(inner_path):
    os.rename(os.path.join(inner_path, file), os.path.join(inner_path, folder + file) )
```
