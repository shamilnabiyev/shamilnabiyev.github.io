---
author: "Shamil Nabiyev"
title: "Resize an image and bbox"
date: "2022-11-17"
tags: [
    "python",
    "computer-vision",
    "augmentation",
    "albumentations"
]
draft: false
---

Resize an image and bounding boxes:

Reference: https://sheldonsebastian94.medium.com/resizing-image-and-bounding-boxes-for-object-detection-7b9d9463125a

```python
import albumentations
from PIL import Image
import numpy as np

sample_img = Image.open("data/img1.jpg")
sample_arr = np.asarray(sample_img)

def resize_image(img_arr, bboxes, h, w):
    """
    :param img_arr: original image as a numpy array
    :param bboxes: bboxes as numpy array where each row is 'x_min', 'y_min', 'x_max', 'y_max', "class_id"
    :param h: resized height dimension of image
    :param w: resized weight dimension of image
    :return: dictionary containing {image:transformed, bboxes:['x_min', 'y_min', 'x_max', 'y_max', "class_id"]}
    """
    # create resize transform pipeline
    transform = albumentations.Compose(
        [albumentations.Resize(height=h, width=w, always_apply=True)],
        bbox_params=albumentations.BboxParams(format='pascal_voc'))

    transformed = transform(image=img_arr, bboxes=bboxes)

    return transformed


transformed_dict = resize_image(sample_arr, bboxes_og, 224, 224)

# contains the image as array
transformed_arr = transformed_dict["image"]

# contains the resized bounding boxes
transformed_info = np.array(list(map(list, transformed_dict["bboxes"]))).astype(float)
```