# Connecting NGINX Server Logs to CloudWatch Logs Using the Command Line

---

## Step 0: Launch an Ubuntu EC2 Instance

1. Go to the AWS Console > EC2 > Launch Instance
2. Choose "Ubuntu Server 22.04 LTS (HVM), SSD Volume Type"
3. Select instance type (t2.micro is Free Tier eligible)
4. Configure instance details:
   - Network: default VPC (or your custom one)
   - Subnet: Choose a public subnet
   - IAM Role: Create or select one with the following policies:
     - AmazonSSMManagedInstanceCore
     - CloudWatchAgentServerPolicy
5. Add storage (keep default or increase based on your needs)
6. Add tags (optional)
7. Configure Security Group:
   - Allow SSH (port 22) from your IP
   - Allow HTTP (port 80) from anywhere
8. Review and launch
9. Download or use an existing key pair to connect via SSH

---

## Connect to Your EC2 Instance

```bash
ssh -i your-key.pem ubuntu@<your-ec2-public-ip>
```

---

## Step 1: Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

You can test that it's working by visiting `http://<your-ec2-public-ip>` in your browser.

---

## Step 2: Install CloudWatch Agent on Ubuntu

### Download the CloudWatch Agent Package

```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
```

### Install the Package

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

## Step 3: Create a Config File for CloudWatch Agent

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste this content:

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

## Step 4: Start the CloudWatch Agent with the Config

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl   -a fetch-config   -m ec2   -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json   -s
```

---

## Step 5: Verify Logs in the AWS Console

1. Go to CloudWatch > Log Groups
2. Look for:
   - `nginx-access-logs`
   - `nginx-error-logs`
3. Click on the log group, then the stream to view incoming log entries

---
