# AWS EC2 Automation: Instance Auditor using Python & Boto3

## 📌 Project Overview
A lightweight Python automation tool developed to programmatically audit, retrieve, and display critical metadata (Instance ID, Lifecycle State, Public/Private IPs, and Tags) of Amazon EC2 instances. The tool runs locally within a secure, isolated Python environment.

---

## 🛠️ Complete Lab Steps

### 1. Secure Local Environment Setup
To comply with modern OS security practices (PEP 668) and prevent corruption of system-wide Linux packages, the project uses an isolated Python virtual environment.

```bash
# Ensure system utilities are complete
sudo apt update && sudo apt install python3-full -y

# Create and activate a secure sandbox
python3 -m venv .venv
source .venv/bin/activate

# Install the official AWS SDK safely inside the environment
pip install boto3
```

### 2. Local Authentication Configuration
AWS credentials are exported as temporary environment variables to ensure zero hardcoding of sensitive access keys inside the source code:

```bash
export AWS_ACCESS_KEY_ID="your_access_key_here"
export AWS_SECRET_ACCESS_KEY="your_secret_key_here"
export AWS_DEFAULT_REGION="us-east-1"
```

### 3. Source Code (`list_instances.py`)
```python
import boto3

def list_ec2_instances():
    # Initialize the high-level object-oriented EC2 resource interface
    ec2 = boto3.resource('ec2')

    print(f"{'Instance ID':<22} | {'State':<12} | {'Public IP':<15} | {'Private IP':<15} | {'Tags'}")
    print("-" * 90)

    # Lazily iterate through all instances across the configured region
    for instance in ec2.instances.all():
        instance_id = instance.id
        state = instance.state['Name']
        public_ip = instance.public_ip_address or "N/A"
        private_ip = instance.private_ip_address or "N/A"
        
        # Parse standard AWS tag Key-Value dictionaries into an optimized string
        if instance.tags:
            tag_list = [f"{tag['Key']}={tag['Value']}" for tag in instance.tags]
            tags_str = ", ".join(tag_list)
        else:
            tags_str = "No Tags"

        print(f"{instance_id:<22} | {state:<12} | {public_ip:<15} | {private_ip:<15} | {tags_str}")

if __name__ == "__main__":
    try:
        list_ec2_instances()
    except Exception as e:
        print(f"An error occurred: {e}")
```

---

## 💡 How to Explain the Script to an Interviewer
*"To prove you wrote this, explain it functionally in under 30 seconds using this structure:"*

> **"I wrote a Python script that leverages the `boto3.resource('ec2')` high-level interface to query the AWS API. It loops through all instances in the region and extracts properties like `instance.id`, `instance.state`, and the networking IPs.**
>
> **Because AWS returns `None` for public IPs if an instance is stopped, I used a fallback mechanism (`or "N/A"`) to prevent code crashes. Lastly, since AWS returns tags as a list of key-value dictionaries, I used a Python list comprehension to gracefully unpack them into a human-readable comma-separated string."**

---

## 🎯 Interview Q&A (Fresher Prep)

### Q1: What is the difference between `boto3.resource()` and `boto3.client()`?
**Answer:** `boto3.client()` is a low-level interface that maps 1:1 directly with the raw AWS HTTP APIs, returning data in complex JSON/dictionary shapes. `boto3.resource()` is a higher-level, object-oriented abstraction. I chose `resource()` for this lab because it represents AWS assets as native Python objects, making operations like `instance.id` cleaner to read and write.

### Q2: Why did you use a Virtual Environment (`venv`) instead of standard `pip install`?
**Answer:** Newer Linux distributions enforce security guidelines (PEP 668) that block global `pip` installations to protect core OS dependencies from breaking. Creating a `venv` builds a localized sandbox. It allowed me to install `boto3` safely without risking the stability of my host system.

### Q3: Why is hardcoding AWS Access Keys inside code bad, and how did you avoid it?
**Answer:** Hardcoding credentials exposes them to severe leaks if code is pushed to version control systems like GitHub. I protected the environment by leveraging Boto3's automated credential lookups, injecting the credentials locally through terminal environment variables.

### Q4: If an instance doesn't have a Public IP, how does your script prevent runtime errors?
**Answer:** Stopped instances or instances in isolated private subnets return `None` for the `.public_ip_address` attribute. If left unhandled, it can corrupt downstream string operations. I resolved this gracefully using short-circuit evaluation: `instance.public_ip_address or "N/A"`.
