# EasyCRUD – Full Stack Application Deployment

EasyCRUD is a **full-stack CRUD (Create, Read, Update, Delete) web application** deployed on **AWS using EC2, Amazon RDS, and Docker containers**.

The backend is built with **Spring Boot (Java)**, the frontend uses **React (Vite)**, and the database runs on **MariaDB hosted on Amazon RDS**.

This project demonstrates a **real-world DevOps deployment workflow including cloud infrastructure, containerization, and service integration.**

---

# Architecture Overview

Frontend (React + Apache) → Backend (Spring Boot API) → Database (Amazon RDS MariaDB)

Infrastructure Components:

- AWS EC2 for application hosting
- Amazon RDS for managed database
- Docker for containerization

---

# Tech Stack

### Cloud
- AWS EC2
- Amazon RDS

### Backend
- Spring Boot (Java)

### Frontend
- React (Vite)
- Node.js

### Database
- MariaDB (Amazon RDS)

### Containerization
- Docker

### Web Server
- Apache (inside frontend container)

---

# Prerequisites

Before starting deployment ensure the following:

- AWS Account
- Ubuntu EC2 Instance
- Amazon RDS (MariaDB)
- Git installed
- Docker installed

### Required Security Group Ports

| Port | Purpose |
|-----|------|
| 22 | SSH access |
| 80 | Frontend |
| 8080 | Backend API |
| 3306 | RDS access (allow only from EC2 Security Group) |

---

# PHASE 1 – Database Setup (Amazon RDS MariaDB)

## Step 1 – Create RDS Database

1. Login to **AWS Console**
2. Navigate to **RDS → Create Database**

Select the following configuration:

Database Engine  
MariaDB

Template  
Free Tier / Production

DB Identifier  
student-db

Master Username and Password  
(Set your credentials)

### Connectivity

- VPC: Same VPC as EC2
- Public Access: Yes (for testing)
- Security Group: Allow **port 3306 from EC2 security group**

Click **Create Database**

After creation copy the **RDS Endpoint**

---

## Step 2 – Launch EC2 Instance

Launch an **Ubuntu EC2 instance**

Connect using SSH:

```bash
ssh -i key.pem ubuntu@<EC2_PUBLIC_IP>
```

Update the instance:

```bash
sudo apt update -y
```

---

## Step 3 – Install MySQL Client

Install MySQL client to connect with RDS.

```bash
sudo apt install mysql-client -y
```

---

## Step 4 – Connect to RDS from EC2

Use the RDS endpoint:

```bash
mysql -h <RDS-ENDPOINT> -u <username> -p
```

Example:

```bash
mysql -h student-db.xxxxx.ap-south-1.rds.amazonaws.com -u admin -p
```

---

## Step 5 – Create Database

Run the following commands inside MySQL:

```sql
SHOW DATABASES;

CREATE DATABASE student_db;

USE student_db;
```

---

## Step 6 – Create Students Table

```sql
CREATE TABLE students (
  id BIGINT NOT NULL AUTO_INCREMENT,
  name VARCHAR(255),
  email VARCHAR(255),
  course VARCHAR(255),
  student_class VARCHAR(255),
  percentage DOUBLE,
  branch VARCHAR(255),
  mobile_number VARCHAR(255),
  PRIMARY KEY (id)
);
```

---

# PHASE 2 – Backend Deployment (Docker)

## Step 7 – Install Docker

Install Docker on EC2:

```bash
sudo apt install docker.io -y
```

Start and enable Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

(Optional)

```bash
sudo usermod -aG docker ubuntu
```

---

## Step 8 – Clone Project Repository

```bash
git clone https://github.com/AnuragPatil-cloud/EasyCRUD-Updated-main.git
```

Navigate to backend directory:

```bash
cd EasyCRUD-Updated/backend
```

---

## Step 9 – Configure Backend Application

Copy the configuration file:

```bash
cp src/main/resources/application.properties .
```

Edit configuration:

```bash
nano application.properties
```

Update database connection:

```properties
server.port=8080

spring.datasource.url=jdbc:mariadb://<RDS-ENDPOINT>:3306/student_db
spring.datasource.username=<username>
spring.datasource.password=<password>

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

## Step 10 – Create Backend Dockerfile

Create a Dockerfile:

```bash
nano Dockerfile
```

Add the following content:

```dockerfile
FROM maven:3.8.5-openjdk-17

COPY . /app
WORKDIR /app

RUN rm -rf src/main/resources/application.properties
RUN cp application.properties src/main/resources/

RUN mvn clean package

WORKDIR /app/target

CMD ["java","-jar","student-registration-backend-0.0.1-SNAPSHOT.jar"]
```

---

## Step 11 – Build Backend Docker Image

```bash
docker build -t backend:v1 .
```

Verify image:

```bash
docker images
```

---

## Step 12 – Run Backend Container

```bash
docker run -d -p 8080:8080 --name backend-container backend:v1
```

Verify container:

```bash
docker ps
```

---

## Step 13 – Verify Backend

Open in browser:

```
http://<EC2_PUBLIC_IP>:8080
```

Backend API should now be running.

---

# PHASE 3 – Frontend Deployment (Docker)

## Step 14 – Navigate to Frontend Directory

```bash
cd ../frontend
```

Show hidden files:

```bash
ls -a
```

---

## Step 15 – Configure Environment File

Edit `.env` file:

```bash
nano .env
```

Update backend API URL:

```env
VITE_API_URL=http://<EC2_PUBLIC_IP>:8080/api
```

---

## Step 16 – Create Frontend Dockerfile

Create Dockerfile:

```bash
nano Dockerfile
```

Add the following content:

```dockerfile
FROM node:18-alpine

COPY . /app
WORKDIR /app

RUN npm install
RUN npm run build

RUN apk add --no-cache apache2

RUN cp -r dist/* /var/www/localhost/htdocs/

EXPOSE 80

CMD ["httpd","-D","FOREGROUND"]
```

---

## Step 17 – Build Frontend Docker Image

```bash
docker build -t frontend:v1 .
```

Verify image:

```bash
docker images
```

---

## Step 18 – Run Frontend Container

```bash
docker run -d -p 80:80 --name frontend-container frontend:v1
```

Verify container:

```bash
docker ps
```

---

## Step 19 – Verify Frontend

Open in browser:

```
http://<EC2_PUBLIC_IP>
```

The **EasyCRUD application UI** should now be accessible.

---

# Project Features

- Full CRUD operations
- Containerized application
- Cloud deployment using AWS
- Backend API built with Spring Boot
- React frontend
- Managed database using Amazon RDS
- Docker-based deployment

---
<img width="1920" height="960" alt="frontend-Easycrud-docker" src="https://github.com/user-attachments/assets/c08fc2eb-3d28-4c74-b717-6160c3070e15" />

---

# Author

**Anurag Patil**  
DevOps Engineer  

GitHub  
https://github.com/AnuragPatil-cloud
