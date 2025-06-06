---
author: "Shamil Nabiyev"
title: "Hpo Optuna"
date: 2025-06-06T21:25:54+02:00
tags: [
    "tag1",
    "tag2"
]
draft: false
---

Hyperparameter optimization for Random Forest Classifier using the Optuna lib

```python
import optuna
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.model_selection import cross_val_score
from sklearn.metrics import classification_report
from sklearn.metrics import precision_recall_fscore_support
from sklearn.metrics import accuracy_score

optuna.logging.set_verbosity(optuna.logging.WARNING)


X, y = make_classification(
    n_samples=250, 
    n_features=10,
    n_informative=5, 
    n_redundant=3,
    random_state=42, 
    shuffle=True
)

X_train, X_test, y_train, y_test = train_test_split(
    X, 
    y, 
    test_size=0.20, 
    random_state=42
)

print(X_train.shape, X_test.shape, y_train.shape, y_test.shape)

def objective(trial):
    # Number of trees in random forest
    n_estimators = trial.suggest_int(name="n_estimators", low=100, high=500, step=100)

    # Number of features to consider at every split
    max_features = trial.suggest_categorical(name="max_features", choices=['auto', 'sqrt']) 

    # Maximum number of levels in tree
    max_depth = trial.suggest_int(name="max_depth", low=10, high=110, step=20)

    # Minimum number of samples required to split a node
    min_samples_split = trial.suggest_int(name="min_samples_split", low=2, high=10, step=2)

    # Minimum number of samples required at each leaf node
    min_samples_leaf = trial.suggest_int(name="min_samples_leaf", low=1, high=4, step=1)
    
    params = {
        "n_estimators": n_estimators,
        "max_features": max_features,
        "max_depth": max_depth,
        "min_samples_split": min_samples_split,
        "min_samples_leaf": min_samples_leaf
    }
    model = RandomForestClassifier(random_state=SEED, **params)
    
    cv_score = cross_val_score(model, X_train, y_train, n_jobs=4, cv=5)
    mean_cv_accuracy = cv_score.mean()
    return mean_cv_accuracy

study = optuna.create_study()
study.optimize(objective, n_trials=5)

# Train a new model using the best parameters
best_model = RandomForestClassifier(random_state=SEED, **study.best_params)
best_model.fit(X_train, y_train)

y_pred = best_model.predict(X_test)

test_acc = accuracy_score(y_test, y_pred)

test_precision, test_recall, test_f1, _ = precision_recall_fscore_support(
    y_test, 
    y_pred, 
    average='binary'
)

print("test_accuracy:", test_acc)
print("test_precision:", test_precision)
print("test_recall:", test_recall)
print("test_f1_score:", test_f1)
```
