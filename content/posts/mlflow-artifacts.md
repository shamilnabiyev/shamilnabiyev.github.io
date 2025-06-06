---
author: "Shamil Nabiyev"
title: "Mlflow artifacts"
date: "2022-09-03"
tags: [
    "python",
    "mlflow"
]
draft: false
---

How to save mlflow runs and artifacts into an external folder 

0. Install the mlflow lib: `pip install mlflow`
1. Create a folder artifacts: `mkdir mlflow-artifacts` 
2. Create a folder sqlite database(s): `mkdir mlflow-dbs`  
3. Start the mlflow UI in terminal: 
    ```bash
    mlflow ui \ 
      --backend-store-uri sqlite:///mlflow-dbs/db-20220822.sqlite \
      --default-artifact-root mlflow-artifacts/
    ```
4. The mlflow UI will be served on `http://127.0.0.1:5000`
5. Create a project folder: `your-project`
6. Switch to the project directory: `cd your-project`
7. Connect to the UI and run experiments:
    ```python
    # experiment.py
    import mlflow
    
    mlflow.set_tracking_uri("http://127.0.0.1:5000")
    mlflow.set_experiment("experiment-001")
    
    with mlflow.start_run():
      mlflow.log_param("num_layers", 5)
      mlflow.log_metric("accuracy", 0.75)
    
    ```
 8. Run the script: `python experiment.py`

 --------------

Autologging contents to an active fluent run, which may be user-created:

```python
import mlflow
import numpy as np
from sklearn import svm
from sklearn import datasets
from sklearn.metrics import f1_score
from sklearn.model_selection import cross_val_score
from sklearn.model_selection import train_test_split

# MLFLow configuration
mlflow.set_tracking_uri("http://127.0.0.1:5000")
mlflow.set_experiment("experiment-002")

# Import the data
X, y = datasets.load_iris(return_X_y=True)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, 
    test_size=0.2, 
    random_state=42
)

# Start mlflow autologging within an active, user-created run
with mlflow.start_run():
    # Enable autologging to log multiple model parameters 
    # and default metrics (training accuracy, training f1 score etc.)
    mlflow.sklearn.autolog(exclusive=False)
    
    # Start model training
    model = svm.SVC(kernel='linear', C=1)
    model.fit(X_train, y_train)
    
    # Calculate f1 score for the test dataset
    y_pred = model.predict(X_test)
    f1_test = f1_score(y_test, y_pred, average="weighted")
    
    # Log additional parameters and metrics manually
    mlflow.log_param("dataset_name", "iris")
    mlflow.log_metric("test_f1_score", f1_test)
    
    # Disable the autologging before closing the current run
    mlflow.sklearn.autolog(disable=True)
```

---------------------

Log additional content (python dict, plots, metrics) into an existing mlflow run:

```python
# import all necessary packages

y_true = tf.concat([y for x, y in test_dataset], axis=0).numpy() 
y_pred = model.predict(test_dataset)

with mlflow.start_run(run_id="a2caa40dee0a2ae18eaedc4f786ec2cb"):
    ##########################################################
    mlflow.log_dict(
        classification_report(
               np.argmax(y_true, axis=1), 
               np.argmax(y_pred, axis=1), 
               digits=4, 
               output_dict=True
        ),
        "classification-report.json"
    )
    
    ##########################################################
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

    ConfusionMatrixDisplay.from_predictions(
        y_true=np.argmax(y_true, axis=1),  
        y_pred=np.argmax(y_pred, axis=1),
        display_labels=np.unique(np.argmax(y_true, axis=1)),
        normalize=None,
        cmap="Blues",
        ax=ax1, 
        colorbar=False,
    )
    ax1.set_title(f"Confusion matrix")

    ConfusionMatrixDisplay.from_predictions(
        y_true=np.argmax(y_true, axis=1),  
        y_pred=np.argmax(y_pred, axis=1),
        display_labels=np.unique(np.argmax(y_true, axis=1)),
        normalize="true",
        cmap="Blues",
        ax=ax2,
        colorbar=False,
    )
    ax2.set_title(f"Confusion matrix (normalized)")
    mlflow.log_figure(fig, "confusion-matrix.png")
    plt.show()
    
    ##########################################################
    
    average_precision = average_precision_score(
        y_true[:,1], 
        y_pred[:,1],
        average="micro"
    )

    print("average_precision:", average_precision)
    mlflow.log_metric("test_average_precision", average_precision)
    
    # Data to plot precision - recall curve
    precision, recall, thresholds = precision_recall_curve(
        y_true[:,1], 
        y_pred[:,1]
    )
    # Use AUC function to calculate the area under the curve of precision recall curve
    auc_precision_recall = auc(recall, precision)
    print("test_pr_auc:", auc_precision_recall)
    mlflow.log_metric("test_pr_auc", auc_precision_recall)
    
    ##########################################################
    mlflow.log_dict({
        "y_true": y_true.tolist(),
        "y_pred": y_pred.tolist()
    }, "ytrue-and-ypred.json")
    
    ##########################################################
    
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(18, 6))
    ax1.set_title("ROC Curve")
    ax2.set_title("Precision Recall Curve")

    roc_disp = RocCurveDisplay.from_predictions(
        y_true=y_true[:,1],
        y_pred=y_pred[:,1],
        name="",
        ax=ax1
    )

    pr_disp = PrecisionRecallDisplay.from_predictions(
        y_true=y_true[:,1],
        y_pred=y_pred[:,1],
        name="",
        ax=ax2
    )
    mlflow.log_figure(fig, "roc-and-pr-curves.png")
    plt.show()
```
