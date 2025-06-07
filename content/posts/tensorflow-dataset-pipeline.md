---
title: "Tensorflow dataset pipeline"
date: "2022-09-02"
tags: [
    "python",
    "tensorflow",
    "tensorflow-dataset"
]
draft: false
---

Useful resources for learning and creating a Tensorflow Dataset. See also code snippets below.


1. Building a data pipeline: https://cs230.stanford.edu/blog/datapipeline/
2. tf.data API, Build TensorFlow input pipelines: https://www.tensorflow.org/guide/data
3. tf.data API, Consuming sets of files: https://www.tensorflow.org/guide/data#consuming_sets_of_files
4. Better performance with the tf.data API: https://www.tensorflow.org/guide/data_performance
5. Keras Sequence Generator: https://stanford.edu/~shervine/blog/keras-how-to-generate-data-on-the-fly
6. Loading and Preprocessing Data with TensorFlow: https://canvas.education.lu.se/courses/3766/pages/chapter-13-loading-and-preprocessing-data-with-tensorflow?module_item_id=109789
7. tf.data.Dataset generators with parallelization: the easy way: https://medium.com/@acordier/tf-data-dataset-generators-with-parallelization-the-easy-way-b5c5f7d2a18

-------------

Feed numpy files (`.npz`) which contain features `X` and a label `y`:

```python
import tensorflow as tf
import numpy as np

def read_npy_file(item):
    x, y = np.load(item.decode())
    return x.astype(np.float32), 

file_list = ['/foo/bar.npz', '/foo/baz.npz']

dataset = tf.data.Dataset.from_tensor_slices(file_list)

dataset = dataset.map(
    lambda item: tuple(
        tf.py_func(func=read_npy_file, inp=[item], Tout=[tf.float32,])
    )
)



# Read numpy files (.npz), extract labels and return a new tf.data.Dataset
def get_dataset(file_names_list, num_classes=2):
    """Creates a new TensorFlow Dataset
    ----------
    Parameters:
        file_names_list: list of file paths
        num_classes: int
    
    Returns:
        (Tensor, Tensor)
    """
    # Load the numpy files
    def map_func(file_path):
        np_data = np.load(file_path)
        x_data = np_data["x"]
        y_label = np_data["y"]
        return x_data.astype(np.float32), tf.one_hot(indices=y_label, depth=num_classes)
    
    # Map function
    numpy_func = lambda item: tf.numpy_function(map_func, [item], [tf.float32, tf.float32])
    
    # Create a new tensorflow dataset
    dataset = tf.data.Dataset.from_tensor_slices(file_list)
    
    # Use map to load the numpy files in parallel
    dataset = dataset.map(numpy_func, num_parallel_calls=tf.data.AUTOTUNE)
    return dataset
```

----------------

The following code snippet has been taken from https://medium.com/@acordier/tf-data-dataset-generators-with-parallelization-the-easy-way-b5c5f7d2a18

```python
import numpy as np
import tensorflow as tf
from custom_utils import create_keras_model


def create_tf_dataset(file_name_list, x_shape, y_shape, batch_size=8, shuffle_dataset=True):
    """Creates a new TensorFlow Dataset for the given list of file names

    Parameter
    --------------
    file_name_list : list, np.array
        A list containing full file paths

    Returns
    --------------
    dataset: tf.data.Dataset
    """

    def processing_func(file_path):
        """Loads the saved numpy array. Returns features and a label"""
        np_data = np.load(file_path)
        x_data = np_data["x"]
        x_data = x_data.astype(np.float32)
        y_label = int(np_data["y"])
        y_label = tf.one_hot(indices=y_label, depth=y_shape[1], dtype=tf.uint8)
        return x_data, y_label

    def func(i):
        i = i.numpy()  # Decoding from the EagerTensor object
        x_data, y_label = processing_func(file_name_list[i])
        return x_data, y_label

    def _fixup_shape(x_data, y_label):
        x_data.set_shape([None, x_shape[1], x_shape[2]])  # n, h, w, c
        y_label.set_shape([None, y_shape[1]])  # n, nb_classes
        return x_data, y_label

    # Data preparation
    z = list(range(len(file_name_list)))

    dataset = tf.data.Dataset.from_generator(lambda: z, tf.uint32)
    
    if shuffle_dataset is True:
        print("shuffling")
        dataset = dataset.shuffle(buffer_size=len(z),
                                  seed=SEED,
                                  reshuffle_each_iteration=True)
    
    dataset = dataset.map(lambda i: tf.py_function(func=func,
                                                   inp=[i],
                                                   Tout=[tf.float32, tf.uint8]),
                          num_parallel_calls=tf.data.AUTOTUNE)
    dataset = dataset.batch(batch_size)
    dataset = dataset.map(_fixup_shape)
    dataset = dataset.prefetch(1)
    return dataset

# Create a list of file paths
training_files = ["full/path/to/file_001.npy", "full/path/to/file_012.npy", ...]
val_files = ["full/path/to/file_502.npy", "full/path/to/file_150.npy", ...]
test_files = ["full/path/to/file_411.npy", "full/path/to/file_590.npy", ...]

train_dataset = create_tf_dataset(training_files)
val_dataset = create_tf_dataset(val_files)
test_dataset = create_tf_dataset(test_files)

model = create_keras_model()

model.fit(x=train_dataset,
          validation_data=val_dataset,
          epochs=100)

model.evaluate(test_dataset, return_dict=True)
```

-----------------

Source: How to extract classes from prefetched dataset in Tensorflow for confusion matrix

**Disclaimer**: the following solution won't work for shuffled Tensorflow datasets (`tensorflow.data.Dataset.shuffle`).

```python
import tensorflow_datasets as tfds
import tensorflow as tf
from sklearn.metrics import confusion_matrix

data, info = tfds.load(
    'iris', 
    split='train',
    as_supervised=True,
    shuffle_files=True,
    with_info=True
)

AUTOTUNE = tf.data.experimental.AUTOTUNE

train_dataset = data.take(120).batch(4).prefetch(buffer_size=AUTOTUNE)
test_dataset = data.skip(120).take(30).batch(4).prefetch(buffer_size=AUTOTUNE)

model = tf.keras.Sequential([
    tf.keras.layers.Dense(8, activation='relu'),
    tf.keras.layers.Dense(16, activation='relu'),
    tf.keras.layers.Dense(info.features['label'].num_classes, activation='softmax')
])

model.compile(loss='sparse_categorical_crossentropy', optimizer='adam', metrics='accuracy')

history = model.fit(train_dataset, validation_data=test_dataset, epochs=50, verbose=0)

y_pred = model.predict(test_dataset)

predicted_categories = tf.argmax(y_pred, axis=1)

true_categories = tf.concat([y for x, y in test_dataset], axis=0)

confusion_matrix(predicted_categories, true_categories)
```
