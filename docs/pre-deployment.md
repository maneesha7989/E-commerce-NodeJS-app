# Pre-Deployment Steps

> [!NOTE]
>
> You should be in your Bastion Server

- Check if Kubernetes Cluster is Connected
```bash
kubectl get nodes
```

### Ingress Nginx

1. Install the Nginx Ingress Controller using Helm:
```bash
kubectl create namespace ingress-nginx
```

2. Add the Nginx Ingress Controller Helm repository:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

3. Install the Nginx Ingress Controller:
```bash
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.service.type=LoadBalancer
```

4. Check the status of the Nginx Ingress Controller:
```bash
kubectl get pods -n ingress-nginx
```

5. Get the external IP address of the LoadBalancer service:
```bash
kubectl get svc -n ingress-nginx
```

### Cert Manager Kubernetes

1. **Jetpack:** Add the Jetstack Helm repository:
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

2. **Cert-Manager:** Install the Cert-Manager Helm chart:
```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.12.0 \
  --set installCRDs=true
``` 

3. **Check pods:**Check the status of the Cert-Manager pods:
```bash
kubectl get pods -n cert-manager
```

You have set up both Cert Manager & Ingress Nginx which are crucial for Application Deployment