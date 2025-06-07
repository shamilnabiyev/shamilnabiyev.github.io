---
title: "Randomized Search CV"
date: "2022-10-31"
tags: [
    "sklearn",
    "python",
    "model-evaluation",
    "hpo"
]
draft: false
---

Hyperparameter tuning for Random Forest Classifier using the RandomizedSearchCV class

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import RandomizedSearchCV
import numpy as np
SEED=42

# Number of trees in random forest
n_estimators = [int(x) for x in range(100,505,100)]
# Number of features to consider at every split
max_features = ['auto', 'sqrt']
# Maximum number of levels in tree
max_depth = [int(x) for x in np.linspace(10, 110, num = 5)]
max_depth.append(None)
# Minimum number of samples required to split a node
min_samples_split = [2, 5, 10]
# Minimum number of samples required at each leaf node
min_samples_leaf = [1, 2, 4]
# Method of selecting samples for training each tree
bootstrap = [True, False]
# Create the random grid
random_grid = {
    'n_estimators': n_estimators,
    'max_features': max_features,
    'max_depth': max_depth,
    'min_samples_split': min_samples_split,
    'min_samples_leaf': min_samples_leaf,
    'bootstrap': bootstrap
}

# Random search of parameters, using 3 fold cross validation, 
# search across 100 different combinations, and use all available cores
rf_random = RandomizedSearchCV(
    estimator=RandomForestClassifier(), 
    param_distributions=random_grid,
    scoring="average_precision",
    random_state=SEED,
    n_iter=10,
    verbose=2,
    n_jobs=4,
    cv=3,
)

# Fit the random search model
rf_random.fit(X_train, y_train)
```
