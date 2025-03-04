# WordPress Deployment on Kubernetes

This repository provides Kubernetes manifests for deploying a WordPress application with a MariaDB database on an AWS EKS cluster.

## Prerequisites
Ensure you have the following installed and configured:

- Kubernetes (Minikube or AWS EKS)
- `kubectl`
- `helm`
- AWS CLI (if using AWS EKS)
- Git

## 1. Clone the Repository
Clone the repository to your local machine:

```sh
git clone https://github.com/NetaAviv/final-project-eks.git
cd final-project-eks
```

## 2. Create a Storage Class (if required)

```sh
kubectl apply -f storage-class.yaml -n [your namespace]
```

## 3. Create Persistent Volume Claims (PVCs)

```sh
kubectl apply -f mysql-pvc-neta.yaml -n [your namespace]
kubectl apply -f wordpress-pvc.yaml -n [your namespace]
```

## 4. Deploy MariaDB

```sh
kubectl apply -f mysql-statefulset.yaml -n [your namespace]
```

## 5. Deploy WordPress

```sh
kubectl apply -f wordpress-deployment.yaml -n [your namespace]
```

## 6. Deploy the Application
There are two options for deployment:

### Option 1: Using Ingress (Recommended)
This method enables easier access to the Grafana dashboard in the future using path-based routing.

```sh
helm install [ingress name] ingress-nginx/ingress-nginx --namespace [your namespace] --set controller.ingressClassResource.name=[ingress class name] -f values.yaml
kubectl apply -f wordpress-service.yaml -n [your namespace]
kubectl apply -f ingress.yaml -n [your namespace]
```

### Option 2: Using LoadBalancer

```sh
echo "  type: LoadBalancer" >> wordpress-service.yaml
kubectl apply -f wordpress-service.yaml -n [your namespace]
```
To view the LoadBalancer details:

```sh
kubectl get service | grep wordpress-service
```

## 7. Verify Deployment
Check the status of your deployed resources:

```sh
kubectl get pods -n [your namespace]
kubectl get services -n [your namespace]
```

## 8. Install Helm for Grafana and Prometheus

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack
```

## 9. Accessing the WordPress Application (If Using Ingress)
Retrieve the external URL of your application:

```sh
kubectl get ingress -n [your namespace]
```
Look for the `HOSTS` column, which will show the URL.

### Open in Your Browser:
- WordPress: `http://host-url/`
- Grafana: `http://host-url/grafana`

