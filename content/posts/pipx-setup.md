---
title: "pipx Setup"
date: "2024-05-30"
tags: [
    "python",
    "venv",
    "pipx"
]
draft: false
---

Install pipx on Windows

1. Run commands in Git Bash
    * `env PIP_REQUIRE_VIRTUALENV=0 python -m pip install --user pipx`
    * `python -m pipx ensurepath --force`
2. Open a new Git Bash terminal
    * `pipx --version`
3. If getting an error during the installation of packages using pipx
    * `No Python at 'C:\Users\<username>\AppData\Local\Programs\Python\PythonX\python.exe'`
    * Then remove the following folder
    * `C:\Users\<username>\.local\pipx`
4. If still having the same issue
    * Then remove the following folder as well
    * `C:\Users\<username>\AppData\Local\pipx`
 