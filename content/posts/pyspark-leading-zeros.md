---
title: "Add leading zeros to PySpark DataFrame"
date: "2024-12-02"
tags: [
    "python",
    "pyspark",
    "data-processing"
]
draft: false
---

PySpark DataFrame - add leading zero if the value is a single digit between 0 and 9 

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, when, regexp_extract, length, lpad

# Create a SparkSession
spark = SparkSession.builder.appName("TupleToDataFrame").getOrCreate()

# List of tuples
arr = [("a", "0a1x"), ("x", "3"), ("cc", "11"), ("h", "5")]

# Create a DataFrame
df = spark.createDataFrame(arr, ["column1", "column2"])

# Function to add leading zero if the value is a single digit between 0 and 9,
# replace non-digit values with "0000", and leave other values as is
df = df.withColumn("column2",
    when(regexp_extract(col("column2"), "^\\d+$", 0) == "", "0000")
    .when((regexp_extract(col("column2"), "^\\d+$", 0) != "") & (length(col("column2")) == 1),
          lpad(col("column2"), 2, "0"))
    .otherwise(col("column2"))
)

# Show the updated DataFrame
df.show()
```
