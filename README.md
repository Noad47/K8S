# Deploying WordPress on Kubernetes

This guide provides Kubernetes manifests for setting up a WordPress application alongside a MariaDB database on an AWS EKS cluster.

## Prerequisites
Before proceeding, ensure you have the following installed and configured:

- Kubernetes (Minikube or AWS EKS)
- `kubectl`
- `helm`
- AWS CLI (for AWS EKS users)
- Git

## 1. Clone the Repository
Download the repository to your local environment:

```sh
git clone https://github.com/NetaAviv/final-project-eks.git
cd final-project-eks
```

## 2. Set Up Storage Class (If Required)

```sh
kubectl apply -f storage-class.yaml -n [namespace]
```

## 3. Create Persistent Volume Claims (PVCs)

```sh
kubectl apply -f mysql-pvc-neta.yaml -n [namespace]
kubectl apply -f wordpress-pvc.yaml -n [namespace]
```

## 4. Deploy the Database (MariaDB)

```sh
kubectl apply -f mysql-statefulset.yaml -n [namespace]
```

## 5. Deploy WordPress

```sh
kubectl apply -f wordpress-deployment.yaml -n [namespace]
```

## 6. Expose the Application
You have two approaches to expose the application:

### Option 1: Using Ingress (Preferred)
Ingress makes it easier to access additional services like Grafana via path-based routing.

```sh
helm install [ingress-name] ingress-nginx/ingress-nginx --namespace [namespace] --set controller.ingressClassResource.name=[ingress-class] -f values.yaml
kubectl apply -f wordpress-service.yaml -n [namespace]
kubectl apply -f ingress.yaml -n [namespace]
```

### Option 2: Using LoadBalancer

```sh
echo "  type: LoadBalancer" >> wordpress-service.yaml
kubectl apply -f wordpress-service.yaml -n [namespace]
```
To check the LoadBalancer details:

```sh
kubectl get service | grep wordpress-service
```

## 7. Validate the Deployment
Check the status of your deployed resources:

```sh
kubectl get pods -n [namespace]
kubectl get services -n [namespace]
```

## 8. Install Prometheus & Grafana via Helm

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install [release-name] prometheus-community/kube-prometheus-stack
```

## 9. Access WordPress (If Using Ingress)
To get the external URL:

```sh
kubectl get ingress -n [namespace]
```
Check the `HOSTS` column to find the application’s URL.

### Open in Your Browser:
- WordPress: `http://your-host-url/`
- Grafana: `http://your-host-url/grafana`

