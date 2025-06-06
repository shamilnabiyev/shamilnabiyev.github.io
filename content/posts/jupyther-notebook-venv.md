---
author: "Shamil Nabiyev"
title: "Add Python Venv to Jupyther Notebook"
date: "2022-04-22"
tags: [
    "python",
    "venv",
    "jupyther"
]
draft: false
---

How to add a python virtual env to jupyther notebook :

1. Activate the virtual environment, e.g. `myvenv`.
2. Install `ipykernel` which provides the IPython kernel for Jupyter
3. Add the virtual environment,`myvenv`, to Jupyter by typing: `python -m ipykernel install --user --name=myvenv`
