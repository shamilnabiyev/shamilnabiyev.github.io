---
title: "Mlflow Quickstart"
date: "2022-05-22"
tags: [
    "python",
    "mlflow",
    "sklearn"
]
draft: false
---

MLFlow Quickstart 

```python
import mlflow
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score, balanced_accuracy_score, f1_score

EXPERIMENT_NAME = 'mlflow-experiment'
EXPERIMENT_ID = mlflow.create_experiment(EXPERIMENT_NAME)

X_train, y_train, X_test, y_test = ...
depth = ...

model = DecisionTreeClassifier(max_depth=depth)
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
acc = accuracy_score(y_test, y_pred)
balanced_acc = balanced_accuracy_score(y_true, y_pred)
f1 = f1_score(y_true)


RUN_NAME = 'run-1'

with mlflow.start_run(experiment_id=EXPERIMENT_ID, run_name=RUN_NAME) as run:
  # Track parameters
  mlflow.log_param('depth', depth)

  # Track metrics
  mlflow.log_metric('accuracy', accuracy)
  mlflow.log_metrics({'balanced_acc': balanced_acc, 'f1': f1})

  # Track model
  mlflow.sklearn.log_model(model, 'dt-classifier')
  
  
# Launch the MLFlow Web UI, accessible at http://localhost:5000
# !mlflow ui

# Retrieve experiment/run results prorammatically and compare them 
client = mlflow.tracking.MlflowClient()

# Retrieve Experiment information
EXPERIMENT_ID = client.get_experiment_by_name(EXPERIMENT_NAME).experiment_id

# Retrieve Runs information (parameter 'depth', metric 'accuracy', 'balanced_accuracy', 'f1-score')
ALL_RUNS_INFO = client.list_run_infos(EXPERIMENT_ID)
ALL_RUNS_ID = [run.run_id for run in ALL_RUNS_INFO]
ALL_PARAM = [client.get_run(run_id).data.params['depth'] for run_id in ALL_RUNS_ID]
ALL_METRIC = [client.get_run(run_id).data.metrics['accuracy'] for run_id in ALL_RUNS_ID]

# View Runs information
df = pd.DataFrame({'run_id': ALL_RUNS_ID, 'params': ALL_PARAM, 'metrics': ALL_METRIC})

# Retrieve Artifact from best run
best_run_id = df.sort_values('metrics', ascending=False).iloc[0]['run_id']
best_model_path = client.download_artifacts(best_run_id, 'dt-classifier')
best_model = mlflow.sklearn.load_model(best_model_path)
```