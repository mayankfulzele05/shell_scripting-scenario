# 🔬 DevOps Lab Setup: Q67 Boto3 Script Hangs

## 📌 Scenario Objective
Diagnose and remediate a production automation script that hangs indefinitely when executed from a private VPC subnet, and enforce resilient network fail-safes.

---

## 📁 Repository Structure
```text
├── .github/workflows/cleanup.yml
├── scripts/
│   └── ec2_cleanup.py
└── requirements.txt
```

---

## 📄 Broken Project Files

### 1. The Automation Script (`scripts/ec2_cleanup.py`)
```python
import os
import sys
import boto3
from botocore.exceptions import ClientError

print("Initializing Boto3 EC2 client...")

# The script attempts to target unmanaged instances in the production region
ec2 = boto3.client('ec2', region_name='us-east-1')

print("Fetching running EC2 instances...")

# Script execution blocks completely at this line below:
response = ec2.describe_instances(
    Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
)

print(f"Found {len(response['Reservations'])} reservations.")
```

---

## 🚨 The Execution Error Log
```text
Initializing Boto3 EC2 client...
Fetching running EC2 instances...

[!] Error: Job timed out after 3600 seconds (1 hour). 
Process terminated violently by the runner manager.
```

---

## 🛠️ Step-by-Step Resolution Blueprint

To fix this hanging automation script permanently, you must apply fixes at both the network layer and the code layer:

1. **Network Layer Fix:** Ensure the environment running the script has a routing path to the AWS API endpoint. In a secure private VPC subnet without public internet routes, you must provision an **AWS VPC Interface Endpoint (PrivateLink)** for EC2. This creates internal, private Elastic Network Interfaces (ENIs) inside your subnet so Boto3 can communicate natively across the AWS internal backbone.
2. **Code Layer Fix (Defensive Coding):** Boto3's default socket timeouts are incredibly loose. If packets drop silently, it can block execution forever. You must apply explicit connection and read timeouts via `botocore.config.Config`.

### 🐍 The Fixed Automation Script (`scripts/ec2_cleanup.py`)
Update the script to enforce absolute network thresholds and graceful error handling:

```python
import os
import sys
import boto3
from botocore.config import Config
from botocore.exceptions import ClientError, ConnectTimeoutError, ReadTimeoutError

print("Initializing resilient Boto3 EC2 client...")

# Define strict timeout limits: 5 seconds to connect, 10 seconds to read data
resilient_config = Config(
    connect_timeout=5,
    read_timeout=10,
    retries={'max_attempts': 2}
)

try:
    ec2 = boto3.client('ec2', region_name='us-east-1', config=resilient_config)
    print("Fetching running EC2 instances...")
    
    response = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )
    
    reservations = response.get('Reservations', [])
    print(f"Success! Found {len(reservations)} reservations.")

except (ConnectTimeoutError, ReadTimeoutError) as timeout_err:
    print(f"🚨 Network Timeout Failure: {timeout_err}", file=sys.stderr)
    print("Ensure VPC Interface Endpoints or NAT Gateways are healthy.", file=sys.stderr)
    sys.exit(1)
except ClientError as api_err:
    print(f"🚨 AWS API Error: {api_err.response['Error']['Message']}", file=sys.stderr)
    sys.exit(1)
```
