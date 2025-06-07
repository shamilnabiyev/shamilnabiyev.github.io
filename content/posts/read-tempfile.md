---
title: "Read tempfile"
date: "2022-07-08"
tags: [
    "python",
    "tempfile"
]
draft: false
---


Read content of file using the `tempfile.NamedTemporaryFile` class.

```python
import tempfile
from sktime.datasets import load_from_tsfile_to_dataframe

tmp_file = tempfile.NamedTemporaryFile(delete=False)
df_tmp = None

try:
    tmp_file.write(content)
    tmp_file.seek(0)

    print(tmp_file.name)
    
    df_tmp = load_from_tsfile_to_dataframe(
        tmp_file.name, 
        return_separate_X_and_y=False
    )
finally:
    tmp_file.close()
    os.unlink(tmp_file.name)
```