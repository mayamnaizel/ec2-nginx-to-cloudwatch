# Connecting NGINX Server Logs to CloudWatch Logs Using the Command Line

---

### Quick Recap of What I’ve Achieved:

Launched and configured an EC2 instance  
Installed and ran NGINX  
Installed and configured the CloudWatch Agent  
Created a custom JSON config to send logs  
Successfully saw the logs in **CloudWatch**!

---

## Correct Way to Install CloudWatch Agent on Ubuntu

Here’s how to do it step-by-step:

---

### Step 1: Download the CloudWatch Agent Package

```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
```

---

### Step 2: Install the Package

```bash
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

### Check if the agent exists

```bash
ls -l /opt/aws/amazon-cloudwatch-agent/bin/
```

You should see files like:

```
amazon-cloudwatch-agent
amazon-cloudwatch-agent-ctl
```

---

## What to Do Next

Now you're ready to:

---

### 1. Create Your Config File

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste in the custom JSON config (with your hardcoded instance ID), then save and exit.

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/nginx/access.log",
            "log_group_name": "nginx-access-logs",
            "log_stream_name": "InstanceID-access"
          },
          {
            "file_path": "/var/log/nginx/error.log",
            "log_group_name": "nginx-error-logs",
            "log_stream_name": "InstanceID-error"
          }
        ]
      }
    }
  }
}
```

---

### 2. Start the Agent with That Config

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

---

### 3. Check in AWS Console

- Go to **CloudWatch > Log groups**
- Look for:
  - `nginx-access-logs`
  - `nginx-error-logs`
- Inside those, you’ll see:
  - `InstanceID-access`
  - `InstanceID-error`

---
