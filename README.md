# Qodeleaf_Assignment

# Step 1: Set Up Your Environment
1. Install Node.js:
2. Install VSCode:
3. Install Postman:

# Step 2: Create the Project
Create a New Project Folder:

mkdir backend-crud
cd backend-crud


# Initialize a Node.js Project:
npm init -y
This creates a package.json file, which will manage your project dependencies.
![alt text](image-2.png)

# Install Express:
npm install express
![alt text](image-3.png)

http://localhost:3000/users
![alt text](image-4.png)

Inside Postman:
Current User:
![alt text](image-5.png)

Add Users:
![alt text](image-6.png)
![alt text](image-7.png)
![alt text](image-10.png)

Delete User:
![alt text](image-8.png)
![alt text](image-9.png)
![alt text](image-11.png)


# Prepare a Dockerfile:

FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]



# Run Locally:

docker build -t backend-app .
![alt text](image-13.png)
![alt text](image-19.png)

docker run -p 3000:3000 backend-app
![alt text](image-14.png)


# AWS CONFIGURE:
![alt text](image-12.png)

# Docker Compose: 
Create a docker-compose.yml to run the backend service with the database locally.

version: "3.8"
services:
  backend:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - database
  database:
    image: postgres:14
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"

![alt text](image-23.png)
![alt text](image-24.png)

# ECR Setup:
![alt text](image-15.png)
![alt text](image-16.png)
![alt text](image-17.png)
![alt text](image-18.png)

# Push the Docker image to ECR:
![alt text](image-20.png)
![alt text](image-21.png)
![alt text](image-22.png)



# 2. CI/CD Configuration
Set up CI pipeline using GitHub Actions.
Build Docker Image on Push: Example GitHub Actions workflow:

name: Build and Push Docker Image
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - name: Build Docker Image
        run: docker build -t username/backend-app:latest .
      - name: Push Docker Image
        run: docker push username/backend-app:latest


# Add the following secrets in GitHub:
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION


# Step 4: Kubernetes Deployment
Create an EKS Cluster:
Use AWS Management Console or Terraform to create an EKS cluster.

provider "aws" {
  region = "us-east-1"
}

module "eks" {
  source = "terraform-aws-modules/eks/aws"
  cluster_name = "backend-app-cluster"
  cluster_version = "1.24"
  subnets = ["subnet-1", "subnet-2"]
  vpc_id = "vpc-12345"
}


# Configure kubectl:
Update kubeconfig to connect to the cluster:

aws eks --region <region> update-kubeconfig --name backend-app-cluster

# Deploy Backend Using Helm:
Create a Helm chart for your application.

Example values.yaml:

image:
  repository: <account-id>.dkr.ecr.<region>.amazonaws.com/backend-app
  tag: latest

service:
  type: LoadBalancer
  port: 3000

deployment:
  replicas: 2


# Deploy with Helm:

helm install backend-app ./chart

# Step 5: Monitoring with Prometheus
Install Prometheus:
Add Prometheus Helm repository:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus:
helm install prometheus prometheus-community/prometheus

# Expose Metrics:
Add metrics collection to the backend (using prom-client for Node.js or prometheus-client for Python).
Create a /metrics endpoint in the backend.

# Update Prometheus Config:
Add a scrape config to values.yaml for Prometheus:

scrape_configs:
  - job_name: backend-app
    static_configs:
      - targets: ['backend-app.default.svc.cluster.local:3000']


# Set Up Alerts:

Add rules in Prometheus:

groups:
  - name: backend-alerts
    rules:
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High Response Time Alert"
      - alert: InstanceDown
        expr: up{job="backend-app"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance is down"


# Integrate Alertmanager:

Configure Alertmanager for Slack, email, or webhook alerts.

# 
# 
