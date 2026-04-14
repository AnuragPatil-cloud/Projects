# 🚀 AI DevOps Assistant (ECS + ECR + Jenkins Deployment)

This project demonstrates how to deploy an **AI DevOps Assistant application** using a complete DevOps pipeline with:

- AWS EC2
- Jenkins (CI/CD)
- Docker
- Amazon ECR
- Amazon ECS (Fargate)
- Application Load Balancer (ALB)

📄 Reference: :contentReference[oaicite:0]{index=0}

---

## 📌 Architecture Overview

1. Launch EC2 (Jenkins Server)
2. Install required tools (Java, Jenkins, Docker, Git)
3. Configure IAM Role
4. Build & Push Docker Image to ECR
5. Deploy using ECS (Fargate)
6. Expose application via Load Balancer

---

## 🛠️ Tech Stack

- AWS (EC2, ECR, ECS, IAM, ALB)
- Jenkins
- Docker
- Git
- Python (Streamlit App)

---

## ⚙️ Setup & Deployment Steps

---

### 🔹 Step 1: Launch EC2 Instance

- AMI: Amazon Linux 2  
- Instance Type: t2.medium  
- Storage: 30 GB  

**Ports:**
- 22 (SSH)
- 8080 (Jenkins)
- 8501 (App)
- 80 (HTTP)

---

### 🔹 Step 2: Create IAM Role

Attach following policies:

- AmazonEC2ContainerRegistryFullAccess  
- AmazonECS_FullAccess  
- AmazonSSMManagedInstanceCore  

Attach role to EC2 instance.

---

### 🔹 Step 3: Connect to EC2

```bash
ssh -i your-key.pem ec2-user@<public-ip>
---
### 🔹 Step 4: Install Java
 ```bash
 sudo yum update -y
sudo yum install java-21-amazon-corretto-headless -y
sudo yum install java-21-amazon-corretto-devel -y
java --version
```
---
###🔹 Step 5: Install Jenkins
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/rpm-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/rpm-stable/jenkins.io-2023.key

sudo yum upgrade -y
sudo yum install jenkins -y

sudo systemctl start jenkins
sudo systemctl enable jenkins
```
Access Jenkins:
http://<EC2-PUBLIC-IP>:8080
---
###🔹 Step 6: Install Docker & Git
```bash
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker ec2-user
sudo usermod -aG docker jenkins

sudo yum install git -y
```
---
###🔹 Step 7: Install Jenkins Plugins
Docker Pipeline
Git
Pipeline
AWS Credentials
Amazon ECR
---
###🔹 Step 8: Create ECR Repository
Example:

<account-id>.dkr.ecr.us-east-1.amazonaws.com/devops-ai
---
###🔹 Step 9: Dockerfile
```bash
FROM python:3.10-slim AS build

WORKDIR /app

RUN rm -rf /var/lib/apt/lists/*
COPY requirements.txt .

RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.10-slim

WORKDIR /app

COPY --from=build /install /usr/local/

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY main.py .

EXPOSE 8501

CMD ["streamlit", "run", "main.py"]
```
---
###🔹 Step 10: Jenkins Pipeline
```bash
pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "<your-ecr-repo>"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: '<your-github-repo>'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops_assistant .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh 'docker tag devops_assistant $ECR_REPO:$IMAGE_TAG'
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }
    }
}
```
---
###🔹 Step 11: Create ECS Cluster (Fargate)
Cluster Name: devops-cluster
Launch Type: Fargate
---
###🔹 Step 12: Task Definition
Task Name: devops-task
CPU: 0.5 vCPU
Memory: 1GB

Container Config:

Image: ECR Image
Port: 8501
---
##3🔹 Step 13: Create ECS Service
Launch Type: Fargate
Desired Tasks: 1
---
###🔹 Step 14: Configure Load Balancer
Type: Application Load Balancer
Listener: HTTP (80)
Target Port: 8501
---
###🔹 Step 15: Access Application

Open in browser:
http://<load-balancer-dns>
---
🔐 Security Groups
ALB:
Allow HTTP (80) from 0.0.0.0/0
ECS Task:
Allow 8501 from ALB Security Group only
---
✅ Final Output
Application deployed on ECS
Accessible via Load Balancer
CI/CD pipeline automated via Jenkins
---
📷 Project Outcome
Fully automated CI/CD pipeline
Scalable container deployment using ECS Fargate
Production-ready architecture
---
📢 Author
Anurag Patil
DevOps Engineer
