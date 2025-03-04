# EKS Workshop: Deploying Apps and Monitoring on Kubernetes

## Overview

This repository contains the resources needed to deploy a WordPress application with a database on an EKS cluster. The deployment includes Kubernetes manifests for services, ingress, and monitoring via Grafana and Prometheus.

## Prerequisites
- Access to an EKS cluster on AWS
- `kubectl` installed and configured
- `Helm` installed
- Amazon ECR repository for storing container images

## Deployment Steps

### 1. Build and Push Docker Image
Pull the WordPress image used in the provided [Docker Compose repository](https://github.com/docker/awesome-compose/blob/master/official-documentation-samples/wordpress/README.md) and push it to Amazon ECR:
```sh
aws ecr create-repository --repository-name wordpress
# Authenticate Docker with AWS ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region>.amazonaws.com
# Pull and push image
docker pull wordpress:latest
docker tag wordpress:latest <aws-account-id>.dkr.ecr.<region>.amazonaws.com/wordpress:latest
docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/wordpress:latest
```

### 2. Install NGINX Ingress Controller
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

### 3. Deploy WordPress Application
```sh
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### 4. Deploy Database with StatefulSet
```sh
kubectl apply -f k8s/database-statefulset.yaml
kubectl apply -f k8s/database-service.yaml
```

### 5. Configure Ingress for External Access
```sh
kubectl apply -f k8s/ingress.yaml
```

### 6. Verify Deployment
```sh
kubectl get pods
kubectl get svc
kubectl get ingress
```

## What I Expect to See in the README
The README should include two deployment options:
- **Deployment with ALB (Application Load Balancer)**
- **Deployment with Ingress Controller**

### Deployment with ALB
To deploy using AWS Load Balancer Controller:
```sh
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    --set clusterName=<your-cluster-name>
```
Apply ALB ingress manifest:
```sh
kubectl apply -f k8s/alb-ingress.yaml
```

### Deployment with Ingress Controller
For deployments using the NGINX Ingress Controller:
```sh
kubectl apply -f k8s/ingress.yaml
```

## Monitoring with Grafana
1. Install Prometheus and Grafana using Helm:
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prom-stack prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```
2. Access Grafana:
```sh
kubectl port-forward svc/kube-prom-stack-grafana 3000:80 -n monitoring
```
3. Create a panel in Grafana to monitor container uptime.

## Definition of Done
- Application is reachable and running
- Grafana monitoring is set up
- Git repository includes all Kubernetes resources using Helm

---
Feel free to modify the README based on your specific needs!
