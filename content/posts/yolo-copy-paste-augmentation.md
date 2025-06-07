---
title: "Copy-Paste Augmentation for YOLO"
date: "2025-04-17"
tags: [
    "pyhon",
    "object-detection",
    "yolo",
    "data-processing",
    "computer-vision"
]
draft: false
---

Simple copy-paste augmentation for YOLO object detection task 

```python
from PIL import Image
import random

def paste_image(source_path, target_path, position=None):
    """
    Pastes a small source image onto a larger target image.

    Parameters:
    - source_path: Path to the small source image.
    - target_path: Path to the larger target image.
    - position: Optional; A tuple (x, y) specifying the top-left corner where the source image will be pasted.
                 If not provided, a random position will be chosen.

    Returns:
    - new_image: The newly created image with the source image pasted onto it.
    - bbox_yolo: A tuple (x_center, y_center, width, height) representing the bounding box of the pasted image in YOLO format.
    """
    # Open the source and target images
    source_image = Image.open(source_path)
    target_image = Image.open(target_path)

    # Get the dimensions of the source and target images
    source_width, source_height = source_image.size
    target_width, target_height = target_image.size

    # Choose a random position if not provided
    if position is None:
        max_x = target_width - source_width
        max_y = target_height - source_height
        if max_x < 0 or max_y < 0:
            raise ValueError("The source image is larger than the target image.")
        position = (random.randint(0, max_x), random.randint(0, max_y))
    else:
        # Ensure the specified position is within bounds
        if position[0] < 0 or position[1] < 0 or position[0] + source_width > target_width or position[1] + source_height > target_height:
            raise ValueError("The specified position is out of the target image bounds.")

    # Paste the source image onto the target image
    target_image.paste(source_image, position)

    # Calculate the bounding box of the pasted image
    left = position[0]
    upper = position[1]
    right = position[0] + source_width
    lower = position[1] + source_height
    bbox = (left, upper, right, lower)

    # Convert the bounding box to YOLO format
    x_center = (left + right) / 2 / target_width
    y_center = (upper + lower) / 2 / target_height
    width = source_width / target_width
    height = source_height / target_height
    bbox_yolo = (x_center, y_center, width, height)

    return target_image, bbox, bbox_yolo

# Example usage
if __name__ == "__main__":
    source_path = "path/to/small_image.png"  # Replace with the path to your small source image
    target_path = "path/to/large_image.png"  # Replace with the path to your large target image

    # Example with a specified position
    new_image, bbox_yolo = paste_image(source_path, target_path, position=(50, 50))
    new_image.save("path/to/new_image_specified_position.png")  # Replace with the desired path to save the new image
    print("Bounding box of the pasted image (specified position) in YOLO format:", bbox_yolo)

    # Example with a random position
    new_image, bbox_yolo = paste_image(source_path, target_path)
    new_image.save("path/to/new_image_random_position.png")  # Replace with the desired path to save the new image
    print("Bounding box of the pasted image (random position) in YOLO format:", bbox_yolo)
```
