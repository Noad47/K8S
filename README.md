# WordPress Deployment on Kubernetes

This repository contains Kubernetes manifests for deploying a WordPress application with a MariaDB database on an AWS EKS cluster.

## Prerequisites
Ensure the following are installed and configured:
- Kubernetes (Minikube or AWS EKS)
- `kubectl`
- `helm`
- AWS CLI (if using AWS EKS)
- Git


## 1. Clone the Repository
Clone the repository to your local machine:

```bash
git clone https://github.com/Noad47/K8S.git
cd final-project-eks
```


## 2. Create a storage class if needed

```bash
kubectl apply -f storage-class.yaml -n [enter your namespace here]
```


## 3. Create PVCs

```bash
kubectl apply -f mysql-pvc-neta.yaml -n [enter your namespace here]
kubectl apply -f wordpress-pvc.yaml -n [enter your namespace here]
```


## 4. Deploy MariaDB



```bash
kubectl apply -f mysql-statefulset.yaml -n [enter your namespace here]
```



## 5. Deploy WordPress



```bash
kubectl apply -f wordpress-deployment.yaml -n [enter your namespace here]
```


## 6. Deploy the Application

### We have two options:
 - Ingress
 - LB

I recommand using ingress so that we could easily access our grafana page in the future using Path-based routing


### Ingress= Set Up Ingress
```bash
helm install [name of your new ingress] ingress-nginx/ingress-nginx --namespace [enter your namespace here] --set controller.ingressClassResource.name=[name your ingress class] -f values.yaml
kubectl apply -f wordpress-service.yaml -n [enter your namespace here]
kubectl apply -f ingress.yaml -n [enter your namespace here]
```
### LB
```bash
echo "  type: LoadBalancer" >> wordpress-service.yaml
kubectl apply -f wordpress-service.yaml -n [enter your namespace here]
```
to view the lb: 
```bash
kubectl get service | grep wordpress-service
```


## 7. Verify Deployment


Check the status of your resources:

```bash
kubectl get pods -n [enter your namespace here]
kubectl get services -n [enter your namespace here]
```


## 8. Install helm for grafana and prometheus:


```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
```bash
helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack
```


## 9. Accessing the WordPress Application- if you used ingress

Retrieve the external URL of your application:

```bash
kubectl get ingress -n [enter your namespace here]
```

Look for the `HOSTS` column, which will show you a URL

In your browser:

- **To view WordPress**: http://host-url/
- **To view Grafana**: http://host-url/grafana



## 10. Cleanup
To delete all resources:

```bash
kubectl delete namespace [enter your namespace here]
```

---
