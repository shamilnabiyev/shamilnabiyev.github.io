---
title: "AWS Glue - read based on date"
date: "2024-09-30"
tags: [
    "aws",
    "glue",
    "pyspark",
    "s3",
    "boto3",
    "python",
    "data-processing"
]
draft: false
---

 Read files from an AWS S3 bucket based on their creation date using AWS Glue 

```python
import boto3
from datetime import datetime
from awsglue.context import GlueContext
from pyspark.context import SparkContext

# First, use boto3 to list and filter objects

def list_s3_files_after_date(bucket_name, prefix='', start_date=None):
    s3 = boto3.client('s3')
    paginator = s3.get_paginator('list_objects_v2')
    page_iterator = paginator.paginate(Bucket=bucket_name, Prefix=prefix)

    filtered_files = []
    for page in page_iterator:
        if 'Contents' in page:
            for obj in page['Contents']:
                # S3 doesn't have a separate creation date, so we use LastModified
                if start_date is None or obj['LastModified'].replace(tzinfo=None) > start_date:
                    filtered_files.append(obj['Key'])

    return filtered_files

# Usage
bucket_name = 'your-bucket-name'
start_date = datetime(2024, 1, 1)  # Files created after January 1, 2024
filtered_files = list_s3_files_after_date(bucket_name, start_date=start_date)

# Then, use AWS Glue to create a DynamicFrame from the filtered files

def read_filtered_files_to_dynamicframe(bucket_name, filtered_files):
    sc = SparkContext()
    glueContext = GlueContext(sc)
    
    if filtered_files:
        dyf = glueContext.create_dynamic_frame.from_options(
            connection_type="s3",
            connection_options={
                "paths": [f"s3://{bucket_name}/{key}" for key in filtered_files],
                "recurse": True,
                "useS3ListImplementation": True
            },
            format="csv",  # Adjust this based on your file format
            format_options={
                "withHeader": True,
                "separator": ","
            }
        )
        return dyf
    else:
        print("No files found matching the criteria.")
        return None

# Usage
bucket_name = 'your-bucket-name'
start_date = datetime(2024, 1, 1)  # Files created after January 1, 2024
filtered_files = list_s3_files_after_date(bucket_name, start_date=start_date)
dyf = read_filtered_files_to_dynamicframe(bucket_name, filtered_files)

if dyf:
    # Print the schema of the DynamicFrame
    dyf.printSchema()
    
    # Convert to DataFrame for further processing if needed
    df = dyf.toDF()
    
    # Perform your manipulations here
    
    # Write the result back to S3 if needed
    glueContext.write_dynamic_frame.from_options(
        frame=dyf,
        connection_type="s3",
        connection_options={"path": "s3://output-bucket/output-path/"},
        format="parquet"
    )
```
