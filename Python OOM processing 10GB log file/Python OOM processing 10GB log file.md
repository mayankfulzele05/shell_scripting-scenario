# 🔬 DevOps Lab Setup: Q68 Python OOM Log Analyzer

## 📌 Scenario Objective
Fix a high-priority production metrics script that crashes with system exit code `137` (OOM Killer) when processing massive application log files, and transition it to a lazy, stream-based architecture.

---

## 📁 Repository Structure
```text
├── scripts/
│   └── analyze_logs.py
└── metrics.json
```

---

## 📄 Broken Project Files

### 1. The Automation Script (`scripts/analyze_logs.py`)
```python
import os
import sys
import json

log_path = "/var/log/nginx/access_huge.log"  # A 10 GB production log file

print("Loading log file into memory...")

# This line causes the Linux kernel OOM Killer to trigger:
with open(log_path, "r") as f:
    log_data = f.read()

print("Processing log data...")
error_count = 0
for line in log_data.split("\n"):
    if " 500 " in line:
        error_count += 1

print(f"Total internal server errors: {error_count}")
```

---

## 🚨 The System Kernel Execution Log
```text
Loading log file into memory...
/bin/bash: line 1: 14205 Killed                  python scripts/analyze_logs.py
[!] System Exit Code: 137 (OOM Killer Invoked)
```

---

## 🛠️ Step-by-Step Resolution Blueprint

To remediate this failure permanently, you must switch from an **in-memory buffer** strategy to a **streaming/lazy execution** architecture:

1. **Avoid Global Reads:** Never use `.read()` or `.readlines()` on files whose maximum potential scale is unknown or exceeds system memory boundaries.
2. **Leverage File Iterators:** In Python, a file descriptor object opened inside a context manager is a built-in generator. Iterating over it with a `for line in f:` loop reads only one line into memory at a time, allowing a file of 10 GB (or even 1 TB) to be processed using just kilobytes of RAM.

### 🐍 The Fixed Automation Script (`scripts/analyze_logs.py`)
Update the script to stream file contents line-by-line using Python's modern iterable design:

```python
import os
import sys

log_path = "/var/log/nginx/access_huge.log"

if not os.path.exists(log_path):
    print(f"🚨 Target log file missing: {log_path}", file=sys.stderr)
    sys.exit(1)

print("Starting memory-efficient log streaming...")
error_count = 0
line_count = 0

# The file object acts as a generator, keeping memory overhead constant
with open(log_path, "r", encoding="utf-8", errors="ignore") as f:
    for line in f:
        line_count += 1
        
        # Process data on-the-fly without accumulating lines in an array
        if " 500 " in line:
            error_count += 1
            
        # Optional: Log progress milestones for transparency
        if line_count % 1_000_000 == 0:
            print(f"Processed {line_count} lines smoothly...")

print("\n--- Analytics Summary ---")
print(f"Total Lines Parsed: {line_count}")
print(f"Total HTTP 500 Failures: {error_count}")
```
