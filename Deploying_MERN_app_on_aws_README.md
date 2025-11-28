## MERN Travel Memory - AWS Infrastructure & Deployment
  
Complete infrastructure-as-code solution for deploying the Travel Memory MERN application on AWS using Terraform and Ansible.

  #Architecture Overview
```
┌─────────────────────────────────────────────────────────┐
│                      AWS VPC                            │
│                                                         │
│  ┌──────────────────────┐  ┌─────────────────────────┐  │
│  │   Public Subnet      │  │   Private Subnet        │  │
│  │                      │  │                         │  │
│  │  ┌───────────────┐  │  │  ┌──────────────────┐   │   │
│  │  │  Web Server   │  │  │  │  MongoDB Server  │   │   │
│  │  │  (EC2)        │──┼──┼──│  (EC2)           │   │   │
│  │  │  - Node.js    │  │  │  │  - MongoDB       │   │   │
│  │  │  - Express    │  │  │  │  - Port 27017    │   │   │
│  │  │  - React      │  │  │  └──────────────────┘   │   │
│  │  └───────────────┘  │  │            │            │   │
│  │         │            │  │           │            │   │
│  └─────────┼────────────┘  └───────────┼────────────┘   │
│            │                           │                │
│     Internet Gateway              NAT Gateway           │
│            │                            │               │
└────────────┼────────────────────────────┼─────────────  ┘
             │                            │
          Internet                    Internet

```  
  Prerequisites

```
AWS Account with appropriate permissions
AWS CLI configured (aws configure)
Terraform >= 1.0
Ansible >= 2.9
SSH key pair for EC2 access
Git
```
  
Project Structure
```
.
├── terraform/
│   ├── main.tf                 # Main infrastructure configuration
│   ├── variables.tf            # Input variables
│   ├── outputs.tf              # Output values
│   ├── vpc.tf                  # VPC and networking
│   ├── security_groups.tf      # Security group rules
│   ├── ec2.tf                  # EC2 instance definitions
│   └── iam.tf                  # IAM roles and policies
├── ansible/
│   ├── inventory/
│   │   └── aws_ec2.yml         # Dynamic inventory
│   ├── playbooks/
│   │   ├── webserver.yml       # Web server configuration
│   │   ├── database.yml        # MongoDB setup
│   │   └── deploy_app.yml      # Application deployment
│   ├── roles/
│   │   ├── nodejs/             # Node.js installation role
│   │   ├── mongodb/            # MongoDB installation role
│   │   └── security/           # Security hardening role
│   └── ansible.cfg             # Ansible configuration
├── scripts/
│   └── deploy.sh               # Automated deployment script
└── README.md
``` 
  
#Part 1: Infrastructure Setup with Terraform

  Step 1: Configure AWS Credentials
``` 
  aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: us-east-1
# Default output format: json

  Step 2: Initialize Terraform
bashcd terraform
terraform init
Step 3: Configure Variables
Create terraform.tfvars:
hclaws_region          = "ca-central-1"
project_name        = "travel-memory"
vpc_cidr            = "10.0.0.0/16"
public_subnet_cidr  = "10.0.1.0/24"
private_subnet_cidr = "10.0.2.0/24"
my_ip               = "YOUR_IP_ADDRESS/32"  # Get from: curl ifconfig.me
key_name            = "your-ec2-key-pair"
instance_type       = "t2.micro"
  
Step 4: Review Infrastructure Plan
bashterraform plan
Step 5: Deploy Infrastructure
bashterraform apply
Resources Created:

VPC with DNS support
Public subnet (10.0.1.0/24)
Private subnet (10.0.2.0/24)
Internet Gateway
NAT Gateway with Elastic IP
Route tables (public and private)
Security groups:

Web server: ports 22 (SSH), 80 (HTTP), 3000 (Node.js), 3001 (React)
Database server: port 27017 (MongoDB, private only)


IAM roles for EC2 instances
2 EC2 instances (Ubuntu 22.04 LTS)

Step 6: Retrieve Outputs

  terraform output
# web_server_public_ip = "54.xxx.xxx.xxx"
# db_server_private_ip = "10.0.2.xxx"

  Part 2: Configuration and Deployment with Ansible

    Step 1: Configure Ansible Inventory

    Create ansible/inventory/aws_ec2.yml:

yamlplugin: aws_ec2
regions:
  - ca-central-1
filters:
  tag:Project: travel-memory
  instance-state-name: running
keyed_groups:
  - key: tags.Role
    prefix: role
hostnames:
  - dns-name
compose:
  ansible_host: public_ip_address
Step 2: Test Ansible Connectivity
bashcd ansible
ansible all -m ping -i inventory/aws_ec2.yml
Step 3: Deploy Database Server
bashansible-playbook -i inventory/aws_ec2.yml playbooks/database.yml
Database Playbook Actions:

Install MongoDB 7.0

  Configure MongoDB for remote access

  Create application database and user
Enable MongoDB service
Configure firewall rules

Step 4: Deploy Web Server
bashansible-playbook -i inventory/aws_ec2.yml playbooks/webserver.yml
Web Server Playbook Actions:

Install Node.js 18.x and NPM
Install PM2 process manager
Clone Travel Memory repository
Install application dependencies
Configure environment variables

Step 5: Deploy Application
ansible-playbook -i inventory/aws_ec2.yml playbooks/deploy_app.yml
Deployment Actions:

Set database connection string
Build React frontend
Start Express backend with PM2
Configure Nginx reverse proxy
Enable application auto-start
```
  
Step 6: Security Hardening
  
ansible-playbook -i inventory/aws_ec2.yml playbooks/security.yml

Security Measures:

Configure UFW firewall

Disable root login via SSH

Set up SSH key-only authentication

Install fail2ban for intrusion prevention

Enable automatic security updates

Configure log rotation

Harden MongoDB authentication

Application Configuration
Environment Variables

Create .env file on web server:
```
envNODE_ENV=production
PORT=3000
MONGODB_URI=mongodb://app_user:password@10.0.2.x:27017/travelmemory
JWT_SECRET=your_jwt_secret_here
```
MongoDB Connection
Database server accessible only from web server via private subnet:
```
Host: 10.0.2.x (private IP)
Port: 27017
Database: travelmemory
User: app_user

Accessing the Application

Web Interface: http://<web_server_public_ip>:3000
API Endpoint: http://<web_server_public_ip>:3000/api
SSH Access: ssh -i your-key.pem ubuntu@<web_server_public_ip>
```
Component Interaction Flow

  User Browser 
  
     ↓
  
  React App (Port 3001)
  
     ↓
  
  Express API (Port 3000)
     ↓
  
  MongoDB (Private Subnet: 27017)

React frontend served from Express static files

API calls routed to /api/* endpoints

Express connects to MongoDB via private subnet

MongoDB isolated from internet, accessible only via NAT Gateway

Automated Deployment Script
Use the provided script for one-command deployment:
bash./scripts/deploy.sh

Monitoring and Maintenance
```
Check Application Status
bashssh ubuntu@<web_server_ip> "pm2 status"
View Application Logs
bashssh ubuntu@<web_server_ip> "pm2 logs"
Database Backup
bashansible-playbook -i inventory/aws_ec2.yml playbooks/backup.yml
Application Updates
bashansible-playbook -i inventory/aws_ec2.yml playbooks/deploy_app.yml --tags update
Troubleshooting
Cannot Connect to Web Server
bash# Check security group rules
terraform show | grep -A 20 "web_sg"

```

# Verify instance is running

```
aws ec2 describe-instances --filters "Name=tag:Role,Values=web"
Database Connection Issues
bash# Test connectivity from web server
ssh ubuntu@<web_ip> "nc -zv 10.0.2.x 27017"
```

# Check MongoDB logs
```
ansible -i inventory/aws_ec2.yml database -m shell -a "tail -100 /var/log/mongodb/mongod.log"
Application Won't Start
bash# Check Node.js installation
ansible -i inventory/aws_ec2.yml web -m shell -a "node --version"
```

# Review PM2 error logs
```
ssh ubuntu@<web_ip> "pm2 logs --err --lines 50"
Cost Optimization
Estimated Monthly Cost: ~$25-30

2x t2.micro instances: ~$15
NAT Gateway: ~$10
Data transfer: ~$2-5
```

To reduce costs:

Use t2.nano instances for testing

Stop NAT Gateway when not in use

Use VPC endpoints for AWS services

Cleanup

To destroy all resources:

bashcd terraform

terraform destroy


Warning: This permanently deletes all infrastructure and data.
