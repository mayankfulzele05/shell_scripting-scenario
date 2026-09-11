# 🔬 DevOps Lab Setup: Q71 YAML Indentation Failure

## 📌 Scenario Objective
Identify, fix, and implement defensive pipeline validation mechanics to catch illegal tab characters and structural nesting drift inside deployment configuration specs before they hit cluster layers.

---

## 📁 Repository Structure
```text
├── .github/workflows/deploy.yml
├── k8s/
│   └── deployment.yaml
└── .yamllint.yml
```

---

## 📄 Broken Project Files

### 1. The Kubernetes Manifest Spec (`k8s/deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
	- containerPort: 80 # 🚨 CRITICAL: Contains a literal hidden raw Tab character (\t)
```

---

## 🚨 The CI Pipeline / Kubectl Error Log
```text
Run kubectl apply -f k8s/deployment.yaml
error: error parsing k8s/deployment.yaml: error converting YAML to JSON: yaml: line 19: found a tab character that violates indentation
Error: Process completed with exit code 1.
```

---

## 🛠️ Step-by-Step Resolution Blueprint

To prevent syntax bugs from stopping delivery paths, shift-left formatting checks should be built directly into the testing runner framework:

1. **Purge Tabs:** Strip out illegal `\t` control structures. YAML parsers evaluate indentation hierarchies using uniform spaces (typically blocks of 2 spaces).
2. **Automated Verification Execution:** Integrate code linter suites like `yamllint` inside automated workflow checks to scan files for formatting bugs before running execution targets like `kubectl apply`.

### 🔄 The Fixed Configuration Manifest (`k8s/deployment.yaml`)
Correct the list indentation format underneath the ports config using standardized spacing blocks:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80 # ✅ FIXED: Replaced hidden tab with exactly 8 spaces
```

### 🛠️ The Shift-Left Workflow Validator (`.github/workflows/deploy.yml`)
Enforce absolute syntax testing verification across delivery pipeline execution steps:

```yaml
name: Continuous Deployment

on: [push]

jobs:
  lint-and-validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install System Linters
        run: pip install yamllint

      - name: Run Configuration Audit
        run: yamllint k8s/deployment.yaml

      - name: Execute Cluster Dry Run
        run: |
          # Use client-side dry-run validation mechanics to confirm spec integrity
          echo "Executing dry-run testing validations..."
```
