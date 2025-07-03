# 🧪 AWS E-Commerce Project Documentation

## 🎯 Objective
Provision EC2 instances to deploy a sample e-commerce web application, configure it behind a Load Balancer, set up Auto Scaling based on traffic, monitor the infrastructure using CloudWatch, and send alerts using SNS.

---

## 1️⃣ Security Group Configuration

### ✅ Configured Inbound Rules:
- **Port 22 (SSH)** – for remote access  
- **Port 80 (HTTP)** – for web traffic

### 📸 Screenshot:
`screenshots/security_group.png`

---

## 2️⃣ EC2 Instance Launch & Setup

### ✅ Launch Configuration:
- **OS**: Amazon Linux 2 / Ubuntu  
- **Public Subnet** with Auto-assign Public IP enabled  
- **IAM Role**: Attached with `AmazonEC2RoleforSSM` and `AmazonSNSFullAccess`  
- **Instance Type**: `t2.micro`  
- **Key Pair**: Used for SSH access  

### ✅ Software Setup:
SSH into EC2 using key and run:
```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Welcome to E-Commerce App</h1>" | sudo tee /var/www/html/index.html
```
📸 **Screenshot:**  
![Image](https://github.com/user-attachments/assets/4ad11a8f-7533-44d6-8101-685571132338)
---

## 3️⃣ Deploy the E-Commerce App

### ✅ Deployment:
- The application is served on **Port 80**
- A simple HTML-based landing page confirms successful deployment

📸 **Screenshot:**  
`screenshots/app_running.png`
![Image](https://github.com/user-attachments/assets/0049b011-a34b-409f-a51a-bc33255915f4)
![Image](https://github.com/user-attachments/assets/a9bce9ba-d31a-4622-a34f-4418a57a57cb)
---

## 4️⃣ Application Load Balancer (ALB)

### ✅ ALB Configuration:
- **Type**: Application Load Balancer  
- **Scheme**: Internet-facing  
- **Listeners**: HTTP (Port 80)  
- **Subnets**: Associated with public subnets  
- **Target Group**: Includes both EC2 instances  
- **Health Check Path**: `/`

📸 **Screenshot:**  
`screenshots/alb_setup.png`
![Image](https://github.com/user-attachments/assets/66ac2f5e-50ae-4a69-9329-59fbbc426482)
![Image](https://github.com/user-attachments/assets/27f60f6f-4bea-4638-ac1a-eacc7496b457)
![Image](https://github.com/user-attachments/assets/42bb81a9-01f4-497e-8831-f1095e4ec67b)
---

## 5️⃣ Auto Scaling Group

### ✅ Launch Template:
- Uses same AMI and instance type (`t2.micro`)  
- **User Data script**:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Auto-Scaled E-Commerce App</h1>" > /var/www/html/index.html
```
## ✅ Auto Scaling Group Settings:
![Image](https://github.com/user-attachments/assets/d2ca069a-c901-4892-b461-cf6d6dd78711)

- **Min Size**: 2  
- **Desired Capacity**: 2  
- **Max Size**: 4  
- **Attached to ALB’s Target Group**

### ✅ Scaling Policy:
- **Scale Out**: CPU > 70%  
- **Scale In**: CPU < 30%

📸 **Screenshot:**  
`screenshots/auto_scaling_setup.png`

---

## 6️⃣ CloudWatch Monitoring
![Image](https://github.com/user-attachments/assets/2d21e905-9a2a-4c0c-bfd9-ff20c6aac855)
### ✅ Metrics Collected:
- CPU Utilization  
- Memory Utilization  
- Disk Space  

### ✅ Alarms Created:
- **CPU Utilization > 70%**  
- **Memory Utilization > 80%**  
- **Instance Status Check Failure**

📸 **Screenshot:**  
`screenshots/cloudwatch_alarms.png`
![Image](https://github.com/user-attachments/assets/68943a9e-f789-4c45-86fc-152c6674ef6e)
---

## 7️⃣ SNS Notifications

### ✅ SNS Topic:
- **Name**: `ecomm-alerts`  
- **Email Subscription**: Created and confirmed
- ![Image](https://github.com/user-attachments/assets/05e463a7-faed-4c29-9a1b-7a72747ac2ee)
![Image](https://github.com/user-attachments/assets/4e343624-d8e0-461b-8548-52d74b11e279)

### ✅ Alarm Actions:
- All CloudWatch alarms are linked to the SNS topic  
- Test alert triggered to confirm delivery  

📸 **Screenshot:**  
`screenshots/sns_confirmation.png`
![Image](https://github.com/user-attachments/assets/c2840a50-fb4d-4a0b-b095-4e43abce76b7)
![Image](https://github.com/user-attachments/assets/a6ef7990-949d-4475-82b9-9772a777ed61)
---

## ✅ Final Testing and Validation

- ✅ EC2 instance connectivity verified via SSH  
- ✅ App accessible from browser via ALB DNS  
- ✅ Scaling tested by generating CPU load  
- ✅ Email notifications received from CloudWatch alarms

