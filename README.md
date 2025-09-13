# TravelMemory Full-Stack Deployment (MERN) with AWS Load Balancer & Auto Scaling.

## Introduction
The **Travel Memory** application is a MERN stack project that allows users to store and share travel memories.  
This guide explains how to deploy the frontend (React) and backend (Node.js/Express) on AWS EC2 instances with:

- **Nginx Reverse Proxy**
- **Application Load Balancer (ALB)**
- **Auto Scaling Groups (ASG)**
- **Security Groups**
- **Target Groups**
- Custom Domain Integration via Cloudflare or other available domain name provider like NameCheap etc

**Repository:** [TravelMemory GitHub](https://github.com/SHArahul/TravelMemory)  

---

## Objectives
- Deploy **backend** on Node.js (port 3001) with Nginx reverse proxy.
- Deploy **frontend** on React (port 3000).
- Ensure frontend-backend communication.
- Create multiple instances for frontend & backend.
- Load balance traffic across instances.
- Connect a custom domain via Cloudflare or any other Domain name provider.

---

## Deployment Architecture
**Flow:**
User → Cloudflare DNS → Frontend tg -> AWS ALB (80/443)
├── /api/* → Backend TG (port 3000, Node.js via Nginx)
↓
MongoDB Atlas (Database Layer)


**Key AWS Components:**
- **VPC** with public (ALB) and private (EC2) subnets.
- **Security Groups** for ALB, frontend, backend, and DB.
- **Target Groups** for frontend & backend with health checks.
- **Auto Scaling Groups** for horizontal scaling.

---

## Step-by-Step Deployment

### **1. Backend Configuration**
1. **Launch EC2 Instance** (Ubuntu).
   
2. SSH into instance:
   ssh -i MyKey_rahul.pem ubuntu@your-ec2-ip

Install Node.js, npm, and Git:
sudo apt update
sudo apt install -y nodejs npm git
Clone repo and install dependencies:

git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
npm install
Create .env file:

PORT=3000
MONGO_URI=<mongodb+srv://<username>:<password>@cluster0.mongodb.net/travelmemory?retryWrites=true&w=majority>

Install Nginx:

sudo apt install -y nginx
Configure Nginx reverse proxy:


sudo nano /etc/nginx/sites-available/backend.conf
Add:

nginx
server {
    listen 80;
    server_name backend.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
Enable config:

sudo ln -s /etc/nginx/sites-available/backend.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx
Start backend:

Install pm2 to make sure server runs without interruption 
npm install -g pm2
pm2 start index.js
pm2 startup

2. Frontend & Backend Connection
Edit frontend/src/urls.js:
export const API_URL = "process.env.REACT_APP_BACKEND_URL || "http://localhost:3001";
Save changes and commit if needed.

3. Frontend Deployment
SSH into frontend EC2 instance.

Install Node.js, npm, git, and Nginx:

sudo apt update
sudo apt install -y nodejs npm git nginx
Clone repo and build:

git clone https://github.com/SHArahul/TravelMemory.git
cd TravelMemory/frontend
npm install
npm run build
Configure Nginx to serve React build:

sudo nano /etc/nginx/sites-available/frontend.conf
nginx
server {
    listen 80;
    server_name example.com;

    root /home/ubuntu/TravelMemory/frontend/build;
    index index.html index.htm;

    location / {
        try_files $uri /index.html;
    }
}
Enable config:

sudo ln -s /etc/nginx/sites-available/frontend.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx


4. Scaling & Load Balancing
Create Target Groups:
Frontend TG:

Port: 3000

Health Check: /

Backend TG:

Port: 3000

Health Check: /health

Create Auto Scaling Groups:
Configure ASG:

Min instances: 2

Max: 6

Scaling policy: CPU-based

Create Application Load Balancer:
Listener on 80 & 443.

Routing rules:

/api/* → Backend TG

Default → Frontend TG

Attach SSL certificate via ACM.

5. Domain Setup with Cloudflare
Point your domain’s nameservers to Cloudflare.

In Cloudflare DNS:

CNAME @ → ALB DNS name (for frontend).

##Security Groups
ALB SG: Allow TCP 80, 443 from 0.0.0.0/0.

Frontend SG: Allow TCP 3000 from ALB SG.

Backend SG: Allow TCP 3001 from ALB SG.


##Architecture diagram (created in draw.io).
<img width="1042" height="852" alt="mern app architecture diagram" src="https://github.com/user-attachments/assets/b4bd0161-fb60-4427-87e5-6ed1856a4b34" />

**ScreenShots**
starting from spinning instances to installing npm, nginx,nodejs,pm2 and app validations and server setup

---
<img width="1025" height="717" alt="Mernapp instance creation" src="https://github.com/user-attachments/assets/e5541063-053b-4999-bd3f-9606be4176b6" />

---
<img width="1870" height="777" alt="security group inbound rules for port" src="https://github.com/user-attachments/assets/25d4ee08-38e3-4f2b-a598-d95b2bc69a79" />

---
<img width="1242" height="410" alt="installing nodeJS" src="https://github.com/user-attachments/assets/e065515b-e85d-4553-bfa2-0e933d53a505" />

---
starting up nodejs

---
status of nodejs
<img width="830" height="127" alt="status of nodejs" src="https://github.com/user-attachments/assets/9d7f0a1d-0f00-4bc5-a260-42c8d17323dc" />

---
setting up mongoDB connection at backend
<img width="1753" height="701" alt="setting up mongo db connection" src="https://github.com/user-attachments/assets/4b90a292-6bff-4baf-8677-df439a6d813a" />
<img width="951" height="353" alt="cloning travel memory" src="https://github.com/user-attachments/assets/14afac22-a3e4-4e46-b80b-44ec258c632c" />

---
installing npm 
<img width="801" height="480" alt="installing npm" src="https://github.com/user-attachments/assets/007070f3-3cc8-4756-b79e-7caac48b693a" />
<img width="782" height="466" alt="starting backend at 3001" src="https://github.com/user-attachments/assets/876529de-d7d0-4be3-ab94-f839b289ac48" />

API request validation
<img width="775" height="156" alt="validating backend server at 3001 port" src="https://github.com/user-attachments/assets/cbd863d5-1224-4e94-b880-1e33325c7535" />

---
React URL
<img width="1261" height="107" alt="react url setup" src="https://github.com/user-attachments/assets/acf3c2cf-b870-4113-83c8-f9d84ec38232" />

---
React App frontend
<img width="1796" height="450" alt="react app frontend running at 3000" src="https://github.com/user-attachments/assets/58b60f77-8cd2-4391-aac1-9211a36df264" />
<img width="1433" height="612" alt="travel memory experience" src="https://github.com/user-attachments/assets/1c5ff2e8-c5d7-45f1-b3cf-7eb349505c81" />

---
population exp tab
<img width="1476" height="832" alt="add exp at travel memory" src="https://github.com/user-attachments/assets/07068483-3f29-42c1-9085-ae39b1de65b5" />
<img width="1433" height="612" alt="travel memory experience" src="https://github.com/user-attachments/assets/10e61f6f-1cf3-451f-b71a-387bf23dd924" />

---
Mongo DB collection entry
<img width="1172" height="951" alt="mongo db collection entry" src="https://github.com/user-attachments/assets/891b494b-555d-4be5-a7c0-214b8074401b" />

Author: Rahul Sharma
