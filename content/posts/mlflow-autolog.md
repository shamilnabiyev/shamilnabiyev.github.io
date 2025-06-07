---
title: "Mlflow autolog"
date: "2022-10-31"
tags: [
    "python",
    "autolog",
    "sklearn",
    "model-evaluation"
]
draft: false
---

MLFow with autologging and custom metrics


```python
from sklearn.metrics import precision_recall_fscore_support
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report
from sklearn.datasets import make_classification
from sklearn.metrics import accuracy_score
import mlflow

mlflow.set_tracking_uri("http://127.0.0.1:5000")
mlflow.set_experiment("experiment-001")

# ------------------------------------------------ #

X, y = make_classification(
    n_samples=250, 
    n_features=10,
    n_informative=5, 
    n_redundant=3,
    random_state=42, 
    shuffle=True
)

print(X.shape, y.shape)

X_train, X_test, y_train, y_test = train_test_split(
    X, 
    y, 
    test_size=0.20, 
    random_state=42
)

print(X_train.shape, X_test.shape, y_train.shape, y_test.shape)

# ------------------------------------------------ #

%%time

with mlflow.start_run():
    mlflow.sklearn.autolog(exclusive=False)
    
    n_estimators = 50
    max_depth = 5
    
    mlflow.log_param("max_depth", max_depth)
    mlflow.log_param("n_estimators", n_estimators)
    
    model = RandomForestClassifier(
        random_state=42, 
        max_depth=max_depth,
        n_estimators=n_estimators
    )
    
    model.fit(X_train, y_train)
    
    y_pred = model.predict(X_test)
    # y_proba = model.predict_proba(X_test)
    
    mlflow.log_dict(
        {
            "y_test": [int(x) for x in y_test],
            "y_pred": [int(x) for x in y_pred]
        }, 
        "ytest-ypred.json"
    )
    
    test_acc = accuracy_score(y_test, y_pred)
    
    test_precision, test_recall, test_f1, _ = precision_recall_fscore_support(
        y_test, 
        y_pred, 
        average='binary'
    )
    
    
    mlflow.log_metric("test_accuracy", test_acc)
    mlflow.log_metric("test_precision", test_precision)
    mlflow.log_metric("test_recall", test_recall)
    mlflow.log_metric("test_f1_score", test_f1)
    
    print("test_accuracy:", test_acc)
    print("test_precision:", test_precision)
    print("test_recall:", test_recall)
    print("test_f1_score:", test_f1)
    
    mlflow.sklearn.autolog(disable=True)

```