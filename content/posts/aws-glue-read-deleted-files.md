---
title: "Aws Glue - read deleted files"
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

 Read files marked as deleted from an AWS S3 bucket into a DynamicFrame using boto3 and AWS Glue 

 ```python
 import boto3
from awsglue.context import GlueContext
from pyspark.context import SparkContext
from awsglue.dynamicframe import DynamicFrame

def read_deleted_files_to_dynamicframe(bucket_name, prefix=''):
    # Initialize Glue context
    sc = SparkContext()
    glueContext = GlueContext(sc)
    
    # Initialize S3 client
    s3 = boto3.client('s3')
    
    # List object versions including delete markers
    paginator = s3.get_paginator('list_object_versions')
    page_iterator = paginator.paginate(Bucket=bucket_name, Prefix=prefix)
    
    # Collect keys of deleted files
    deleted_files = []
    for page in page_iterator:
        if 'DeleteMarkers' in page:
            for delete_marker in page['DeleteMarkers']:
                if delete_marker['IsLatest']:
                    deleted_files.append(delete_marker['Key'])
    
    # Read deleted files into a DynamicFrame
    if deleted_files:
        # Create a DynamicFrame from the deleted files
        dyf = glueContext.create_dynamic_frame.from_options(
            connection_type="s3",
            connection_options={
                "paths": [f"s3://{bucket_name}/{key}" for key in deleted_files],
                "recurse": True,
                "useS3ListImplementation": True,
                "readVersion": "LATEST_PREVIOUS"  # Read the version before the delete marker
            },
            format="csv",  # Adjust this based on your file format
            format_options={
                "withHeader": True,
                "separator": ","
            }
        )
        
        return dyf
    else:
        print("No deleted files found.")
        return None

# Usage
bucket_name = 'your-bucket-name'
prefix = 'your-folder-prefix/'  # Optional

dyf = read_deleted_files_to_dynamicframe(bucket_name, prefix)

if dyf:
    # Print the schema of the DynamicFrame
    dyf.printSchema()
    
    # Convert to DataFrame for further processing if needed
    df = dyf.toDF()
    
    # Perform your manipulations here
    
    # Convert back to DynamicFrame if necessary
    result_dyf = DynamicFrame.fromDF(df, glueContext, "result_dyf")
    
    # Write the result back to S3
    glueContext.write_dynamic_frame.from_options(
        frame=result_dyf,
        connection_type="s3",
        connection_options={"path": "s3://output-bucket/output-path/"},
        format="parquet"
    )

 ```
