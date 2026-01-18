
# DCRM – Django CRM on AWS (Solution Design)

This document describes how to use the DCRM (Django CRM learning project) as a small AWS hands-on scenario.

The goal is **not** to build a production system, but to practice:

- deploying a Django app on **AWS EC2**
- using **MySQL** on the server (local on EC2, later RDS)
- deploying a Django app on **AWS EC2** using `runserver` for a simple lab setup (Phase 1)
- optionally extending it later with **Gunicorn + Nginx**, **RDS**, **ALB** and **Auto Scaling** (Phase 2)
- understanding basic AWS networking (VPC, Security Groups, SSH, HTTP)

---

## 1. High-Level Overview

### 1.1. Current State (Local)

- Django application: `DCRM`
- Database: **MySQL** (local)
- Environment: local machine, no deployment
- Purpose: learning Django (auth, CRUD, templates, ORM, MySQL)

### 1.2. Target State (AWS – Phase 1)

- **EC2 instance (Ubuntu)** running:
  - Python + virtualenv
  - Django app (DCRM)
  - MySQL server (local on EC2)
  - Django development server (`runserver`) listening on port `8000`
- Access via browser using the EC2 public IP and port `8000`

> Phase 2 introduces **RDS**, **ALB**, **Auto Scaling** and optionally **Gunicorn + Nginx**.

---

## 2. Architecture (Phase 1 – Simple Deploy)

```text
User Browser
    |
    | HTTP :8000
    v
[ Django runserver (DCRM) ]  (on EC2)
    |
    | local :3306
    v
[ MySQL Database ]            (on EC2)
```

### Networking (simplified)

**Security Group (Phase 1)**

- 22 (SSH)
- 8000 (Django dev server)

**Security Group (Phase 2 – planned)**

- ALB: 80 (HTTP)
- EC2: 8000 (from ALB SG)
- RDS: 3306 (from EC2 SG)

---

# 3. AWS Tasks – Step by Step (Phase 1)

## 3.1. Create an EC2 Instance

- Go to **EC2 → Instances → Launch instance**
- Name: `dcrm-ec2-demo`
- AMI: **Ubuntu Server 22.04 LTS** (or similar)
- Instance type: `t2.micro` (free tier) or `t3.micro`
- Key pair: create or select an existing key pair

Network settings:

- VPC: default
- Subnet: any public subnet
- Auto-assign public IP: **enabled**

Security Group (Phase 1):

- Inbound:
  - SSH (22) from your IP
  - Custom TCP (8000) from anywhere `0.0.0.0/0` (for lab)
- Outbound:
  - allow all

Then: **Launch the instance**.

---

## 3.2. Connect to EC2 and Prepare the System

SSH into EC2:

```bash
ssh -i <your-key>.pem ubuntu@<ec2-public-ip>
```

Update packages:

```bash
sudo apt update && sudo apt upgrade -y
```

Install required packages:

```bash
sudo apt install -y python3-pip python3-venv python3-dev     build-essential libmysqlclient-dev mysql-server git
```

*(Nginx is not needed in Phase 1.)*

---

## 3.3. Clone the Project & Create Virtualenv

```bash
cd /var/www
sudo mkdir dcrm
sudo chown ubuntu:ubuntu dcrm
cd dcrm
```

Clone your repo:

```bash
git clone https://github.com/<your-username>/<your-dcrm-repo>.git .
```

Create and activate virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Install Python dependencies:

```bash
pip install --upgrade pip
pip install -r requirements.txt
# If requirements.txt does not exist:
# pip install django mysqlclient
```

---

## 3.4. Configure MySQL on EC2

Secure MySQL (optional but recommended):

```bash
sudo mysql_secure_installation
```

Login to MySQL:

```bash
sudo mysql -u root -p
```

Create database and user:

```sql
CREATE DATABASE dcrm_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'dcrm_user'@'localhost' IDENTIFIED BY 'strong-password';
GRANT ALL PRIVILEGES ON dcrm_db.* TO 'dcrm_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Update `settings.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dcrm_db',
        'USER': 'dcrm_user',
        'PASSWORD': 'strong-password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

(And make sure `ALLOWED_HOSTS` includes the EC2 public IP.)

Apply migrations and create superuser:

```bash
python manage.py migrate
python manage.py createsuperuser
```

---

## 3.5. Test the App on EC2

Run the development server:

```bash
python manage.py runserver 0.0.0.0:8000
```

Open in your browser:

```text
http://<ec2-public-ip>:8000
```

If you see the DCRM application, **Phase 1 basic deploy works** ✅

From here you have two options:

- keep using `runserver` for a simple learning deployment (**this document**)
- or later move to Gunicorn + Nginx as part of a more production-like setup (**Phase 2**)

For now we keep it **simple** and stay with `runserver`.

---

## 3.6. Keep the App Running (Simple Deploy Without Gunicorn)

For this learning scenario we will keep using:

```bash
python manage.py runserver 0.0.0.0:8000
```

### Security Group Reminder

Keep the inbound rule for port **8000/tcp** in the EC2 Security Group:

- Type: Custom TCP  
- Port: 8000  
- Source: `0.0.0.0/0` (for lab only)

> In a real production setup you would typically use port 80 (HTTP) behind a web server / load balancer.

### Start the app

On the EC2 instance:

```bash
cd /var/www/dcrm
source venv/bin/activate
python manage.py runserver 0.0.0.0:8000
```

Open in your browser:

```text
http://<ec2-public-ip>:8000
```

You should see the DCRM application.

### Optional: keep `runserver` running after logout

For a simple lab you can just keep the SSH session open.  
If you want the server to keep running after logout, you can use `nohup`:

```bash
cd /var/www/dcrm
source venv/bin/activate
nohup python manage.py runserver 0.0.0.0:8000 &
```

This will:

- start Django in the background  
- detach from your SSH session  

To see if it is running:

```bash
ps aux | grep runserver
```

To stop it, find the PID and kill it:

```bash
kill <PID>
```

---

# 4. Advanced AWS Architecture (RDS + ALB + Auto Scaling Group) – Phase 2

The simple EC2 + local MySQL deployment works for learning, but modern cloud deployments separate concerns:

- App servers = stateless & disposable  
- Database = centralized (**RDS**)  
- Load balancing = **ALB**  
- Horizontal scaling = **ASG**  

This section upgrades the architecture toward a more realistic cloud design.

---

## 4.1. New Architecture Overview

```text
User
  |
  v
[ Application Load Balancer ]
            |
        Target Group
            |
     ---------------------
     |        |         |
  [EC2]    [EC2]     [EC2]   <-- Auto Scaling Group
            |
            v
        [ RDS MySQL ]
```

**Key improvements:**

- App servers can scale up/down without losing data  
- DB is centralized (persistent, managed)  
- Stateless execution model  
- No direct user access to EC2  
- ALB health checks & routing  

---

## 4.2. Move Database to Amazon RDS (MySQL)

### Create RDS instance

- Engine: MySQL (free tier eligible)  
- Instance class: `db.t3.micro`  
- Storage: `gp2` or `gp3`  
- VPC: default VPC  
- Public access: **No**  
- Multi-AZ: optional / off for lab  

Credentials:

- DB name: `dcrm_db`  
- User: `dcrm_user`  
- Password: `<strong-password>`  

### Networking

RDS Security Group inbound:

- Type: MySQL/Aurora  
- Port: 3306  
- Source: **EC2 Security Group**  

> Do **not** open port 3306 to the internet.

### Update Django settings

In `settings.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dcrm_db',
        'USER': 'dcrm_user',
        'PASSWORD': 'strong-password',
        'HOST': '<rds-endpoint>',
        'PORT': '3306',
    }
}
}
```

Migrate:

```bash
python manage.py migrate
```

At this point, multiple EC2 instances can share the same DB.

---

## 4.3. Make Application Servers Stateless

To allow scaling:

- No local DB (RDS handles persistence)  
- Avoid local file uploads  
- Static files should be shared (e.g. S3)  

Optional static configuration for later:

```bash
AWS_STORAGE_BUCKET_NAME=<bucket>
```

This is not required to continue but recommended for a real cloud app.

---

## 4.4. Create Launch Template for EC2

**Purpose:** new EC2 instances should boot with the same app configuration.

Options for bootstrapping app code:

- `git clone` from GitHub in user-data  
- AMI baked via Packer (advanced)  
- ECR container (Docker/ECS path)  

For simplicity, user-data example:

```bash
#!/bin/bash
apt update -y
apt install -y python3-venv python3-pip git mysql-client
cd /var/www
git clone https://github.com/<user>/dcrm.git
cd dcrm
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
nohup python manage.py runserver 0.0.0.0:8000 &
```

This makes each EC2 capable of self-provisioning.

---

## 4.5. Create Auto Scaling Group (ASG)

ASG config:

- Launch Template → created earlier  
- VPC → default  
- Subnets → public subnets  
- Desired capacity: 1  
- Min size: 1  
- Max size: 3  
- Health check type: EC2 or ALB  
- Cooldown: default  

Scaling policies (optional):

- CPU ↑ → scale out  
- CPU ↓ → scale in  

---

## 4.6. Add Application Load Balancer (ALB)

Create ALB:

- Scheme: **Internet-facing**  
- Type: **Application Load Balancer**  
- Listener:
  - HTTP :80 → Target Group  

Target Group:

- Target type: Instance  
- Health checks:
  - Path: `/`  
  - Protocol: HTTP  
  - Port: 8000 (for this lab)  

Attach ASG to this Target Group.

Routing:

Users hit ALB DNS:

```text
http://<ALB-DNS>
```

---

## 4.7. Final Networking Model

**ALB Security Group**

- Inbound:
  - 80 from `0.0.0.0/0`
- Outbound:
  - allow all

**EC2 Security Group**

- Inbound:
  - 8000 from **ALB Security Group**
- Outbound:
  - allow all

**RDS Security Group**

- Inbound:
  - 3306 from **EC2 Security Group**
- Outbound:
  - allow all

---

## 4.8. Resulting Behavior

With this setup:

- Users always connect to **ALB**  
- ALB routes traffic to healthy EC2 instances  
- ASG adds/terminates EC2 nodes dynamically  
- RDS persists data independently of scaling  
- No data loss on EC2 termination  
- Resilience improves significantly  

> Optional: In Phase 2 the Django `runserver` can be replaced with **Gunicorn** behind **Nginx** or directly behind the ALB, which is a more production-like deployment pattern.

---

# 5. Optional Enhancements

- Replace `runserver` → **Gunicorn + Nginx**  
- Move static files → **S3**  
- Add **CloudFront** for CDN  
- Add **ACM** for HTTPS via ALB  
- Add **IAM Roles** (access S3 / SSM)  
- Add **CloudWatch** monitoring & logs  
- Use **Terraform** or **CloudFormation** to codify infrastructure  

---

# 6. Summary

This solution evolves a small local Django + MySQL learning project into a scalable AWS architecture in two phases:

- **Phase 1:** Simple learning deploy using EC2 + local MySQL + `runserver`
- **Phase 2:** Cloud-native architecture using **ALB + ASG + RDS**

The goal is not production readiness but to build hands-on AWS experience with:

- EC2  
- RDS  
- Load balancing  
- Auto scaling  
- Security Groups  
- Cloud networking  
- Stateless application design
