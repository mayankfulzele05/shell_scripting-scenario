# 🔬 DevOps Lab Setup: Q66 Python ModuleNotFoundError in CI

## 📌 Scenario Objective
Fix a broken GitHub Actions workflow pipeline that crashes during an automated deployment task due to path, installation, and naming mismatches.

---

## 📁 Repository Structure
```text
├── .github/workflows/deploy.yml
├── scripts/
│   └── deploy_infra.py
└── requirements.txt
```

---

## 📄 Broken Project Files

### 1. The CI Pipeline Configuration (`.github/workflows/deploy.yml`)
```yaml
name: Continuous Deployment
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Run Deploy Script
        run: python scripts/deploy_infra.py
```

### 2. The Python Automation Script (`scripts/deploy_infra.py`)
```python
import os
import sys
import pyyaml  # Intended to parse configuration files
import requests

print("Starting infrastructure deployment...")
# Logic for deploying infra goes here
```

### 3. The Dependency Manifest (`requirements.txt`)
```text
requests==2.31.0
PyYAML==6.0.1
```

---

## 🚨 The CI Pipeline Error Log
```text
Run python scripts/deploy_infra.py
Traceback (most recent call last):
  File "/home/runner/work/app/scripts/deploy_infra.py", line 3, in <module>
    import pyyaml
ModuleNotFoundError: No module named 'pyyaml'
Error: Process completed with exit code 1.
```

---

## 🛠️ Step-by-Step Resolution Blueprint

To fix this pipeline, you must address two major issues:
1. **Missing Step:** The workflow configures Python but never executes `pip install` to load dependencies into the fresh runner environment.
2. **Case-Sensitivity Mismatch:** Python import modules are strictly case-sensitive on Linux environments. While the package distribution name is `PyYAML`, it must be imported as `import yaml` inside the script.

### 🔄 The Fixed Configuration (`.github/workflows/deploy.yml`)
Add a dependency installation step **before** running the deployment tool:

```yaml
name: Continuous Deployment
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run Deploy Script
        run: python scripts/deploy_infra.py
```

### 🐍 The Fixed Automation Script (`scripts/deploy_infra.py`)
Correct the package import casing to match the official `PyYAML` registry specifications:

```python
import os
import sys
import yaml  # FIX: Changed from 'import pyyaml' to standard case-sensitive 'import yaml'
import requests

print("Starting infrastructure deployment...")
```
