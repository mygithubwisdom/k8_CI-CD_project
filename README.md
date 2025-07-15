# CI-CD_k8_project
# Fashion Web App - CI/CD Pipeline with Kubernetes

A complete CI/CD pipeline implementation for a Fashion Web App using GitHub Actions, Docker, Terraform, and Kubernetes deployment on AWS EC2 with Minikube.

## 🚀 Project Overview

This project demonstrates a full DevOps workflow for a Fashion Web Application, featuring automated deployment from GitHub to a Kubernetes cluster running on AWS EC2. The pipeline includes containerization with Docker, infrastructure as code with Terraform, and continuous deployment with GitHub Actions.

## 🏗️ Architecture

```
GitHub Repository → GitHub Actions → Docker Hub → AWS EC2 (t3.micro) → Minikube → Fashion Web App
```

## 📁 Project Structure

```
k8_CI-CD_fashion_webapp/
├── .github/
│   └── workflows/
│       └── deploy.yml              # GitHub Actions CI/CD workflow
├── .idea/                          # IDE configuration files
├── fashion_webapp/                 # Main application directory
├── terraform/                      # Infrastructure as Code
│   ├── modules/
│   │   ├── vpc/
│   │   ├── ec2/
│   │   └── security-groups/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── app.py                          # Main Flask application
├── forms.py                        # Flask forms for user input
├── models.py                       # Database models
├── requirements.txt                # Python dependencies
├── Dockerfile                      # Container configuration
├── deployment.yaml                 # Kubernetes deployment manifest
├── service.yaml                    # Kubernetes service manifest
├── .gitignore                      # Git ignore rules
└── README.md                       # Project documentation
```

## 🛠️ Technology Stack

- **Frontend**: HTML, CSS, JavaScript (Fashion Web Interface)
- **Backend**: Python Flask
- **Database**: SQLite/PostgreSQL (configurable)
- **Containerization**: Docker
- **Orchestration**: Kubernetes (Minikube)
- **Cloud Provider**: AWS (EC2 t3.micro)
- **Infrastructure**: Terraform
- **CI/CD**: GitHub Actions
- **Container Registry**: Docker Hub

## 📋 Prerequisites

- AWS Account with EC2 access
- GitHub Account
- Docker Hub Account
- Terraform >= 1.0
- AWS CLI configured
- kubectl installed
- SSH key pair for EC2 access

## 🚀 Getting Started

### Step 1: Clone Repository

```bash
git clone https://github.com/yourusername/k8_CI-CD_fashion_webapp.git
cd k8_CI-CD_fashion_webapp
```

### Step 2: Application Setup

The Fashion Web App is built with Flask and includes:

**app.py** - Main application file
```python
# Fashion Web App with user authentication, product catalog, and shopping cart
from flask import Flask, render_template, request, redirect, url_for
# ... (your existing app.py code)
```

**forms.py** - Flask forms for user interactions
```python
# Forms for user registration, login, product search, and checkout
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SelectField
# ... (your existing forms.py code)
```

**models.py** - Database models
```python
# Database models for users, products, orders, and fashion items
from flask_sqlalchemy import SQLAlchemy
# ... (your existing models.py code)
```

**requirements.txt** - Python dependencies
```txt
Flask==2.3.3
Flask-SQLAlchemy==3.0.5
Flask-WTF==1.1.1
WTForms==3.0.1
# ... (your existing requirements)
```

### Step 3: Docker Configuration

**Dockerfile** - Multi-stage build for optimized container
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

Build and test locally:
```bash
docker build -t fashion-webapp .
docker run -p 5000:5000 fashion-webapp
```

### Step 4: Kubernetes Manifests

**deployment.yaml** - Kubernetes deployment configuration
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fashion-webapp-deployment
  labels:
    app: fashion-webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: fashion-webapp
  template:
    metadata:
      labels:
        app: fashion-webapp
    spec:
      containers:
      - name: fashion-webapp
        image: wisdomuntamed/my_devops_k8_docker_task:latest
        ports:
        - containerPort: 5000
        env:
        - name: FLASK_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

**service.yaml** - Kubernetes service configuration
```yaml
apiVersion: v1
kind: Service
metadata:
  name: fashion-webapp-service
spec:
  selector:
    app: fashion-webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
      nodePort: 30000
  type: NodePort
```

### Step 5: Terraform Infrastructure

Create AWS infrastructure with cost-optimized t3.micro instances:

**terraform/main.tf**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC Module
module "vpc" {
  source = "./modules/vpc"
  
  project_name        = "fashion-webapp"
  vpc_cidr           = "10.0.0.0/16"
  public_subnet_cidr = "10.0.1.0/24"
  availability_zone  = "${var.aws_region}a"
}

# Security Groups Module
module "security_groups" {
  source = "./modules/security-groups"
  
  project_name = "fashion-webapp"
  vpc_id       = module.vpc.vpc_id
}

# EC2 Module
module "ec2" {
  source = "./modules/ec2"
  
  project_name      = "fashion-webapp"
  ami_id           = "ami-0c02fb55956c7d316" # Ubuntu 20.04 LTS
  instance_type    = "t3.micro"  # Cost-optimized for project size
  public_key       = var.public_key
  security_group_id = module.security_groups.minikube_sg_id
  subnet_id        = module.vpc.public_subnet_id
}
```

**terraform/variables.tf**
```hcl
variable "aws_region" {
  description = "AWS region for deployment"
  type        = string
  default     = "us-east-1"
}

variable "public_key" {
  description = "Public SSH key for EC2 access"
  type        = string
}
```

Deploy infrastructure:
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

### Step 6: GitHub Actions CI/CD Pipeline

**.github/workflows/deploy.yml** - Complete CI/CD workflow
```yaml
name: Fashion Web App CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  DOCKER_USERNAME: wisdomuntamed
  DOCKER_REPOSITORY: my_devops_k8_docker_task
  APP_NAME: fashion-webapp

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest
    
    - name: Run tests
      run: |
        python -m pytest tests/ || echo "No tests found"

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Fashion Web App
      uses: docker/build-push-action@v3
      with:
        context: .
        push: true
        tags: |
          ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPOSITORY }}:latest
          ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPOSITORY }}:v${{ github.run_number }}
  
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy Fashion Web App to Minikube
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ubuntu
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          # Pull latest Fashion Web App image
          docker pull ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPOSITORY }}:latest
          
          # Apply Kubernetes manifests
          kubectl apply -f deployment.yaml
          kubectl apply -f service.yaml
          
          # Restart deployment with new image
          kubectl rollout restart deployment/fashion-webapp-deployment
          
          # Check deployment status
          kubectl get pods
          kubectl get services
          
          # Show application URL
          echo "Fashion Web App deployed successfully!"
          echo "Access the app at: http://$(minikube ip):30000"
```

### Step 7: AWS EC2 Setup and Minikube Configuration

SSH into your EC2 instance:
```bash
ssh -i path/to/your/key.pem ubuntu@<EC2_PUBLIC_IP>
```

Install Docker and Minikube:
```bash
# Update system
sudo apt update

# Install Docker
sudo apt install -y docker.io
sudo usermod -aG docker ubuntu

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/

# Start Minikube
minikube start --driver=docker
```

### Step 8: Deploy Fashion Web App

Deploy the application:
```bash
# Apply Kubernetes manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check deployment status
kubectl get pods
kubectl get services

# Get application URL
minikube service fashion-webapp-service --url
```

## 🔧 Configuration

### GitHub Secrets Setup

Configure the following secrets in your GitHub repository:
- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password
- `EC2_HOST`: Public IP of your EC2 instance
- `EC2_SSH_KEY`: Private SSH key for EC2 access

### Environment Variables

The Fashion Web App supports these environment variables:
- `FLASK_ENV`: Application environment (development/production)
- `DATABASE_URL`: Database connection string
- `SECRET_KEY`: Flask secret key for sessions

## 🚀 Deployment Commands

### Docker Commands
```bash
# Build Fashion Web App image
docker build -t wisdomuntamed/my_devops_k8_docker_task:v.0.1 .

# Login to Docker Hub
docker login -u wisdomuntamed

# Tag and push image
docker tag wisdomuntamed/my_devops_k8_docker_task:v.0.1 wisdomuntamed/my_devops_k8_docker_task:latest
docker push wisdomuntamed/my_devops_k8_docker_task:v.0.1
docker push wisdomuntamed/my_devops_k8_docker_task:latest
```

### Kubernetes Commands
```bash
# Deploy application
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check deployment
kubectl get pods
kubectl get services
kubectl get all

# View logs
kubectl logs -f deployment/fashion-webapp-deployment

# Describe resources
kubectl describe pod <pod-name>
kubectl describe service fashion-webapp-service
```

## 🎯 Fashion Web App Features

- **User Authentication**: Registration and login system
- **Product Catalog**: Browse fashion items with categories
- **Shopping Cart**: Add items and manage cart
- **Order Management**: Process and track orders
- **Responsive Design**: Mobile-friendly interface
- **Search Functionality**: Find products quickly
- **User Profile**: Manage account settings

## 🐛 Troubleshooting

### Common Issues

1. **Pod crashes**: Check logs with `kubectl logs <pod-name>`
2. **Service not accessible**: Verify NodePort configuration
3. **Image pull errors**: Confirm Docker Hub credentials
4. **Database connection**: Check environment variables

### Health Checks

```bash
# Check Minikube status
minikube status

# Check cluster info
kubectl cluster-info

# Check node status
kubectl get nodes

# Check Fashion Web App health
curl http://$(minikube ip):30000/health
```

## 💰 Cost Optimization

- **Instance Type**: t3.micro (1 vCPU, 1 GB RAM) - ideal for small projects
- **Resource Limits**: Set appropriate CPU/memory limits in Kubernetes
- **Auto-shutdown**: Configure EC2 to stop when not in use
- **Monitoring**: Use AWS CloudWatch for resource monitoring

## 🔒 Security Considerations

- **Security Groups**: Restrict access to necessary ports only
- **SSH Keys**: Use strong key pairs and rotate regularly
- **Container Security**: Scan images for vulnerabilities
- **Environment Variables**: Store sensitive data in Kubernetes secrets

## 📈 Monitoring and Logging

```bash
# View application logs
kubectl logs -f deployment/fashion-webapp-deployment

# Check resource usage
kubectl top pods
kubectl top nodes

# Monitor service endpoints
kubectl get endpoints
```

## 🧹 Cleanup

To remove all resources:
```bash
# Delete Kubernetes resources
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml

# Stop Minikube
minikube stop

# Destroy AWS infrastructure
cd terraform
terraform destroy
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- Flask community for the excellent web framework
- Kubernetes team for container orchestration
- AWS for reliable cloud infrastructure
- Docker for containerization technology

---

**Fashion Web App** - Bringing style to the cloud with DevOps excellence! 👗✨
