# Qodeleaf_Assignment

# Step 1: Set Up Your Environment
1. Install Node.js:
2. Install VSCode:
3. Install Postman:

# Step 2: Create the Project
Create a New Project Folder:
```bash
mkdir backend-crud
cd backend-crud
```

# Initialize a Node.js Project:
```bash
npm init -y
```
This creates a package.json file, which will manage your project dependencies.
![image](https://github.com/user-attachments/assets/8b9efcdb-36bf-45d3-991c-8db39b5068dc)


# Install Express:
```bash
npm install express
```
![image](https://github.com/user-attachments/assets/3125865a-9463-4b35-bd1b-6f0c3811f066)


```bash
http://localhost:3000/users
```
![image](https://github.com/user-attachments/assets/9640ae43-36e3-4c25-bde7-7900a9d2064b)


Inside Postman:
Current User:
![image](https://github.com/user-attachments/assets/e9400b6e-462d-4fdc-af3d-a84c321b39e7)

Add Users:
![image](https://github.com/user-attachments/assets/e44d0f5c-ef3e-4941-b4b0-c68629b0816d)

![image](https://github.com/user-attachments/assets/af4c4903-b9e8-4fcd-9bb3-1bcf09f3e844)

![image](https://github.com/user-attachments/assets/990b3a10-db53-416e-84ce-cb84b39856b2)


Delete User:
![image](https://github.com/user-attachments/assets/5d946287-fcbc-41f2-8abb-b0bbe69f0d1c)

![image](https://github.com/user-attachments/assets/df33b716-998d-4478-8ade-ca0f72ee19d3)

![image](https://github.com/user-attachments/assets/09811df9-8d23-41dd-a1a8-5b0b491d6bac)



# Prepare a Dockerfile:
```bash
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```


# Run Locally:
```bash
docker build -t backend-app .
```
![image](https://github.com/user-attachments/assets/4597547c-78d2-47d0-ac47-fca21dc0e9de)

![image](https://github.com/user-attachments/assets/690bb9d0-cca4-4c65-a655-40ab4283d61f)


```bash
docker run -p 3000:3000 backend-app
```
![image](https://github.com/user-attachments/assets/3941fcd8-e625-4fa2-9251-b21998bcef65)



# AWS CONFIGURE:
![image](https://github.com/user-attachments/assets/cdb02029-d294-40c8-b195-322880bdf293)


# Docker Compose: 
Create a docker-compose.yml to run the backend service with the database locally.
```bash
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
```
![image](https://github.com/user-attachments/assets/bbf1302d-1ffe-408c-bd74-4a642c7e26c4)

![image](https://github.com/user-attachments/assets/2ea8f488-2bfc-4f14-b0da-852283a909df)


# ECR Setup:
![image](https://github.com/user-attachments/assets/6316f7df-3709-41a9-837b-5d8aca4493f1)

![image](https://github.com/user-attachments/assets/f1e54370-09c1-4285-b871-4dbfeffd949f)

![image](https://github.com/user-attachments/assets/e2a86eec-2ff7-4ae3-b739-616aae687c33)

![image](https://github.com/user-attachments/assets/39aaf636-0b1a-40b3-9ddc-e8ec08060c9b)


# Push the Docker image to ECR:
![image](https://github.com/user-attachments/assets/3912faa4-1228-4ad5-8be3-fe3550656c5b)

![image](https://github.com/user-attachments/assets/d8910ae8-f90e-4650-9283-5cdfaf29d261)
![image](https://github.com/user-attachments/assets/26ce1383-46de-48c2-8d47-0497f603353b)




# 2. CI/CD Configuration
Set up CI pipeline using GitHub Actions.
Build Docker Image on Push: Example GitHub Actions workflow:
```bash
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
```

# Add the following secrets in GitHub:
```bash
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
```


# Step 4: Kubernetes Deployment
Create an EKS Cluster:
Use AWS Management Console or Terraform to create an EKS cluster.
```bash
# provider block for AWS
provider "aws" {
  region = "us-east-1"
}

# Create VPC for the EKS cluster
resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "eks-vpc"
  }
}

# Create subnets for the EKS cluster
resource "aws_subnet" "eks_subnet_a" {
  vpc_id = aws_vpc.eks_vpc.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "eks-subnet-a"
  }
}

resource "aws_subnet" "eks_subnet_b" {
  vpc_id = aws_vpc.eks_vpc.id
  cidr_block = "10.0.2.0/24"
  availability_zone = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "eks-subnet-b"
  }
}

# Security Group for the EKS cluster
resource "aws_security_group" "eks_security_group" {
  vpc_id = aws_vpc.eks_vpc.id

  egress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
  }

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
  }

  tags = {
    Name = "eks-sg"
  }
}

# Create the EKS Cluster
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "backend-app-cluster"
  cluster_version = "1.24"
  subnets         = [aws_subnet.eks_subnet_a.id, aws_subnet.eks_subnet_b.id]
  vpc_id          = aws_vpc.eks_vpc.id

  node_groups = {
    eks_nodes = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1

      instance_type = "t3.medium"
      key_name      = "my-ec2-key"  # Replace with your EC2 Key Pair name
    }
  }

  node_security_group = aws_security_group.eks_security_group.id
}

# Output the EKS cluster details
output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_id" {
  value = module.eks.cluster_id
}

output "cluster_name" {
  value = module.eks.cluster_name
}

```
```bash
terraform init
terraform plan
terraform apply
```

||  OR  ||
```bash
eksctl create cluster --version 1.28 --region ap-south-1 --name proj-eks --nodes 2 --nodes-min 2 --nodes-max 2 --nodegroup-name proj-node-group --managed
kubectl get nodes
kubectl get pods -n kube-system
```
![image](https://github.com/user-attachments/assets/45185bd1-8d28-43c9-a275-67cba480fdd1)
![image](https://github.com/user-attachments/assets/9d0a9ec7-4d4c-40ef-89e8-f10dde4d9340)
![image](https://github.com/user-attachments/assets/63180e05-6429-4fc1-b901-547c89da0396)



# Configure kubectl:
Update kubeconfig to connect to the cluster:
```bash
aws eks --region <region> update-kubeconfig --name backend-app-cluster
```
# Deploy Backend Using Helm:
Create a Helm chart for your application.

![image](https://github.com/user-attachments/assets/9db90bd5-da7e-414f-941d-4adff1d9dfd7)



Example values.yaml:
```bash
image:
  repository: <account-id>.dkr.ecr.<region>.amazonaws.com/backend-app
  tag: latest

service:
  type: LoadBalancer
  port: 3000

deployment:
  replicas: 2
```

# Deploy with Helm:
```bash
helm install backend-app ./chart
```

![image](https://github.com/user-attachments/assets/8e6947dd-2153-4a0c-a955-c24513338979)

# Step 5: Monitoring with Prometheus
Install Prometheus:
Add Prometheus Helm repository:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

![image](https://github.com/user-attachments/assets/c3ca77be-51c2-4823-89e0-9814c54179ae)

# Install Prometheus:
```bash
helm install prometheus prometheus-community/prometheus
```
# Expose Metrics:
Add metrics collection to the backend (using prom-client for Node.js or prometheus-client for Python).
Create a /metrics endpoint in the backend.

# Update Prometheus Config:
Add a scrape config to values.yaml for Prometheus:
```bash
scrape_configs:
  - job_name: backend-app
    static_configs:
      - targets: ['backend-app.default.svc.cluster.local:3000']
```

# Set Up Alerts:

Add rules in Prometheus:
```bash
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
```

# Integrate Alertmanager:

Configure Alertmanager for Slack, email, or webhook alerts.

# 
# 
