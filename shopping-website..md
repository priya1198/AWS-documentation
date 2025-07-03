# 🧪 AWS Project: Deploy & Scale E-Commerce App (Using Default VPC Only)

## 🎯 Objective
Deploy a Node.js-based e-commerce web application using EC2, Load Balancer, Auto Scaling Group, CloudWatch monitoring, and SNS alerts — all using AWS **Default VPC** (no custom VPC creation required).

---

## ✅ 1️⃣ Provision EC2 Infrastructure (in Default VPC)

### 📌 Step 1: Use Default VPC and Public Subnets
No action required — AWS provides:
- 1 subnet per Availability Zone
- Automatically assignable public IPs
- Internet Gateway attached

### 📌 Step 2: Create a Security Group
**Go to**: EC2 > Security Groups > Create Security Group  
- **Name**: `ecomm-sg`  
- **Inbound Rules**:
  - SSH (port 22) → Your IP
  - HTTP (port 80) → 0.0.0.0/0  
- **Outbound Rule**: Default (allow all)

### 📌 Step 3: Create IAM Role
**Go to**: IAM > Roles > Create Role  
- **Trusted Entity**: EC2  
- **Attach Policies**:
  - `AmazonEC2RoleforSSM`
  - `CloudWatchAgentServerPolicy`
  - `AmazonSNSFullAccess`  
- **Role Name**: `EcommEC2Role`

### 📌 Step 4: Launch Two EC2 Instances
**Go to**: EC2 > Launch Instance  
- AMI: Amazon Linux 2 or Ubuntu  
- Instance Type: `t2.micro`  
- Network: **Default VPC**  
- Subnet: Choose two different AZs (e.g., `us-east-1a`, `us-east-1b`)  
- Auto-assign Public IP: **Enabled**  
- IAM Role: `EcommEC2Role`  
- Security Group: `ecomm-sg`

> ⚠️ Launch two instances manually or automate later via Launch Template.

### 📌 Step 5: Install Node.js App
SSH into each EC2 instance and run:

```bash
# Update & install dependencies
sudo yum update -y
curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs git
```
# Clone sample Node.js app
```
git clone https://github.com/heroku/node-js-sample.git
cd node-js-sample
npm install
PORT=80 sudo node index.js &
```
---
## ✅ 2️⃣ Deploy the E-Commerce App

- Repeat the setup on **both EC2 instances** as done in Step 1.
- Ensure that the app is listening on **port 80**.
- You can verify by accessing the instance's public IP or using `curl http://localhost`.

---

## ✅ 3️⃣ Configure Application Load Balancer (ALB)

### 📌 Step 1: Create Target Group

**Navigate to**: EC2 > Target Groups > Create Target Group  
- **Type**: Instance  
- **Protocol**: HTTP  
- **Port**: 80  
- **VPC**: Default VPC  
- **Health Check Path**: `/`  
- **Name**: `ecomm-tg`

➡️ Add both EC2 instances as targets after creation.

---

### 📌 Step 2: Create ALB

**Navigate to**: EC2 > Load Balancers > Create Load Balancer  
- **Type**: Application Load Balancer  
- **Name**: `ecomm-alb`  
- **Scheme**: Internet-facing  
- **Network**: Default VPC  
- **Subnets**: Choose 2 different public subnets  
- **Security Group**: Must allow HTTP (port 80)

**Listener Rule**:
- Forward HTTP traffic on port 80 to target group `ecomm-tg`

🔍 **Test**: Copy the ALB **DNS name** and open it in your browser — your app should load.

---

## ✅ 4️⃣ Set Up Auto Scaling

### 📌 Step 1: Create Launch Template

**Navigate to**: EC2 > Launch Templates > Create  
- **Name**: `ecomm-launch-template`  
- **AMI**: Amazon Linux 2 or Ubuntu  
- **Instance Type**: `t2.micro`  
- **IAM Role**: `EcommEC2Role`  
- **Security Group**: `ecomm-sg`  
- **User Data**: Add the following startup script:

```bash
#!/bin/bash
yum update -y
curl -sL https://rpm.nodesource.com/setup_18.x | bash -
yum install -y nodejs git
git clone https://github.com/heroku/node-js-sample.git
cd node-js-sample
npm install
PORT=80 nohup node index.js > app.log 2>&1 &
```
## 📌 Step 2: Create Auto Scaling Group

**Navigate to**: EC2 > Auto Scaling > Create Auto Scaling Group

- **Name**: `ecomm-asg`
- **Launch Template**: Use `ecomm-launch-template`
- **Network**: Default VPC
- **Subnets**: Select 2 public subnets
- **Attach to Target Group**: `ecomm-tg`

### Scaling Configuration:
- **Minimum capacity**: 2  
- **Desired capacity**: 2  
- **Maximum capacity**: 4

---

## 📌 Step 3: Configure Scaling Policies

**Policy Based on CPU Utilization**:

- **Scale Out**: When CPU > 70%  
- **Scale In**: When CPU < 30%

---

## ✅ 5️⃣ Enable Monitoring via CloudWatch

### 📌 Step 1: Install CloudWatch Agent

SSH into **one EC2 instance**, then run the following commands:

```bash
# Install the agent
sudo yum install amazon-cloudwatch-agent -y

# Create CloudWatch configuration
cat <<EOF | sudo tee /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
{
  "metrics": {
    "append_dimensions": {
      "InstanceId": "\${aws:InstanceId}"
    },
    "metrics_collected": {
      "CPU": {
        "measurement": ["cpu_usage_idle"],
        "metrics_collection_interval": 60
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      }
    }
  }
}
EOF

# Start the CloudWatch agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config -m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```
## 📌 Step 2: Create CloudWatch Alarms

**Navigate to**: CloudWatch > Alarms > Create Alarm

Create the following alarms:

- **HighCPUAlarm**: Trigger when **CPU Utilization > 70%**
- **HighMemoryAlarm**: Trigger when **Memory Utilization > 80%**
- **InstanceStatusAlarm**: Trigger on **EC2 Status Check Failure**

---

## ✅ 6️⃣ Set Up SNS Notifications

### 📌 Step 1: Create SNS Topic

**Navigate to**: SNS > Topics > Create Topic

- **Name**: `ecomm-alerts`  
- **Type**: Standard

---

### 📌 Step 2: Subscribe Your Email

- **Protocol**: Email  
- **Enter your email address**  
- **Confirm** the subscription via the confirmation email in your inbox

---

### 📌 Step 3: Attach SNS to CloudWatch Alarms

- For each **CloudWatch alarm**, go to **"Add Notification Action"**
- Choose **SNS Topic**: `ecomm-alerts`

---



