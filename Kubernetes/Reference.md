# Amazon Prime Video Clone — DevSecOps Deployment on AWS EKS

A hands-on DevSecOps project that deploys an Amazon Prime Video Clone built with ReactJS on AWS EKS, using a fully automated and secure CI/CD pipeline.

---

## Project Overview

This project demonstrates a complete DevSecOps workflow for deploying a React-based Amazon Prime Video Clone on AWS Elastic Kubernetes Service (EKS). Security is integrated at every stage of the pipeline — from code commit to production deployment.

---

## Application Stack

- **Frontend:** HTML, CSS, JavaScript, ReactJS
- **Runtime:** Node.js
- **Containerization:** Docker
- **Orchestration:** AWS EKS (Kubernetes)

---

## DevSecOps Pipeline Stages

| Stage | Tool | Purpose |
|-------|------|---------|
| Source Control | GitHub | Code repository |
| CI/CD | Jenkins | Automated pipeline |
| Code Quality | SonarQube | Static code analysis |
| Quality Gate | SonarQube | Block bad code from proceeding |
| Dependency Scan | OWASP Dependency Check | Find vulnerable libraries |
| File System Scan | Trivy | Scan files for vulnerabilities |
| Image Build | Docker | Containerize the application |
| Image Scan | Docker Scout | Scan container image for CVEs |
| Image Registry | DockerHub | Store and version Docker images |
| GitOps | ArgoCD | Sync Kubernetes deployments from Git |
| Orchestration | AWS EKS | Run containers at scale |
| Monitoring | Prometheus + Grafana | Metrics and dashboards |
| Notifications | Email (SMTP) | Build status alerts |

---

## Infrastructure Setup

### Prerequisites
- AWS Account with IAM permissions
- EC2 instance (t2.large recommended) for Jenkins server
- AWS CLI configured
- kubectl installed
- eksctl installed
- Terraform (optional, for IaC)

### Tools Installed on Jenkins Server
- Jenkins
- Docker
- SonarQube (Docker container)
- Trivy
- Docker Scout
- kubectl
- AWS CLI

---

## Jenkins Pipeline

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage("Clean Workspace")       { steps { cleanWs() } }
        stage("Git Checkout")          { steps { git branch: 'main', url: 'https://github.com/juhisinha422/Prime-Video-Clone-Deployment.git' } }
        stage("SonarQube Analysis")    { ... }
        stage("Quality Gate")          { ... }
        stage("Install Dependencies")  { steps { sh "npm install" } }
        stage("OWASP Dependency Check"){ ... }
        stage("Trivy File Scan")       { steps { sh "trivy fs . > trivy.txt" } }
        stage("Build Docker Image")    { steps { sh "docker build -t amazon-prime ." } }
        stage("Tag & Push to DockerHub"){ ... }
        stage("Docker Scout Scan")     { ... }
        stage("Deploy to Container")   { steps { sh "docker run -d -p 3000:3000 amazon-prime" } }
    }
}
```

---

## EKS Cluster Setup

```bash
# Create EKS cluster
eksctl create cluster \
  --name=prime-video-cluster \
  --region=us-east-1 \
  --nodes=2 \
  --node-type=t3.medium \
  --managed

# Update kubeconfig
aws eks update-kubeconfig \
  --name prime-video-cluster \
  --region us-east-1

# Verify nodes
kubectl get nodes
```

---

## ArgoCD Setup on EKS

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose ArgoCD UI
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'

# Get ArgoCD URL
kubectl get svc argocd-server -n argocd

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

---

## Monitoring Setup

```bash
# Add Helm repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

# Get Grafana password
kubectl get secret --namespace monitoring prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d
```

---

## Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: amazon-prime
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: amazon-prime
  template:
    metadata:
      labels:
        app: amazon-prime
    spec:
      containers:
      - name: amazon-prime
        image: juhisinha/amazon-prime:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: amazon-prime-svc
spec:
  type: LoadBalancer
  selector:
    app: amazon-prime
  ports:
  - port: 80
    targetPort: 3000
```

---

## Jenkins Plugins Required

- Git Plugin
- SonarQube Scanner
- OWASP Dependency Check
- Docker Pipeline
- Docker Scout
- NodeJS Plugin
- Email Extension Plugin
- Kubernetes CLI Plugin

---

## Security Tools Summary

| Tool | What It Scans |
|------|--------------|
| SonarQube | Source code quality & vulnerabilities |
| OWASP Dependency Check | Third-party library CVEs |
| Trivy | Filesystem & Docker image vulnerabilities |
| Docker Scout | Container image CVEs & recommendations |

---

## Cleanup

```bash
# Delete EKS cluster
eksctl delete cluster --name=prime-video-cluster --region=us-east-1

# Remove ArgoCD
kubectl delete namespace argocd

# Remove Monitoring
kubectl delete namespace monitoring

# Stop Jenkins EC2 instance from AWS Console
```

---

## Repository

- **App Source Code:** https://github.com/juhisinha422/Prime-Video-Clone-Deployment
- **Reference:** Deploying Amazon Prime Video Clone on AWS EKS Using DevSecOps Approach
- https://www.youtube.com/watch?v=uaiuUGg5gLE&t=5545s
