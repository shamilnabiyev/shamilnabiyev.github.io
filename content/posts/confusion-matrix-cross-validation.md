---
author: "Shamil Nabiyev"
title: "Confusion matrix for cross validation"
date: "2022-11-02"
tags: [
    "python",
    "cross-validation",
    "mlflow",
    "numpy",
    "sklearn"
]
draft: false
---

Confusion matrix for cross validation. Results are being saved as mlflow artifacts and retrieved later for evaluation purposes. 

```python
import mlflow
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import precision_recall_fscore_support
from sklearn.metrics import accuracy_score
from sklearn.metrics import confusion_matrix
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import StratifiedKFold


SEED = 42
mlflow.set_tracking_uri("http://127.0.0.1:5000")
mlflow.set_experiment("cross-validation-average")

# Data preparation
dataset = load_breast_cancer()
X = dataset.data
y = dataset.target

print(X.shape, y.shape)

# Model training
skf = StratifiedKFold(n_splits=5, random_state=SEED, shuffle=True)

for train_index, test_index in skf.split(X, y):
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
    
    with mlflow.start_run():
        mlflow.sklearn.autolog(exclusive=False)
        model = RandomForestClassifier(random_state=SEED)
        model.fit(X_train, y_train)
        
        y_pred = model.predict(X_test)
        mlflow.log_dict({"y_test": [int(x) for x in y_test],
                         "y_pred": [int(x) for x in y_pred]
                        }, "ytest-ypred.json")
        
        test_acc = accuracy_score(y_test, y_pred)
        mlflow.log_metric("test_accuracy", test_acc)
        print("test_accuracy:", test_acc)

        test_precision, test_recall, test_f1, _ = precision_recall_fscore_support(
            y_test, 
            y_pred, 
            average='binary'
        )
        mlflow.log_metric("test_precision", test_precision)
        mlflow.log_metric("test_recall", test_recall)
        mlflow.log_metric("test_f1_score", test_f1)
        
        tn, fp, fn, tp = confusion_matrix(y_test, y_pred).ravel()
        mlflow.log_metric("tn", tn)
        mlflow.log_metric("fp", fp)
        mlflow.log_metric("fn", fn)
        mlflow.log_metric("tp", tp)
        
        tn, fp, fn, tp = confusion_matrix(
            y_test, 
            y_pred, 
            normalize="true"
        ).ravel()

        mlflow.log_metric("tn_normalized", tn)
        mlflow.log_metric("fp_normalized", fp)
        mlflow.log_metric("fn_normalized", fn)
        mlflow.log_metric("tp_normalized", tp)
        
        mlflow.sklearn.autolog(disable=True)
  
# Results evaluation
runs = mlflow.search_runs(experiment_ids=["2"])

columns = [
    'metrics.tn_normalized', 'metrics.fp_normalized', 
    'metrics.fn_normalized', 'metrics.tp_normalized'
]

mean_confusion_matrix = runs[columns].mean()
print(mean_confusion_matrix)
```
Output:

<center>
    <img src="/img/confusion-matrix-for-cross-validation.png" alt="confusion-matrix-for-cross-validation" width="800"/>
</center>
