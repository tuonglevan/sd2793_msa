# Introduction
This repository contains the code and resources needed to build and deploy the Assignment Microservice Architecture.
## Features:
- Jenkins Pipeline.
- Azure Pipeline.
- Backend Application.
- Frontend Application.
- Deployment to EKS and AKS.
- CD by ArgoCD.
- Monitoring by Prometheus and Grafana.

# Installation
## ArgoCD
### Step 1: Install ArgoCD
1. **Create the namespace for ArgoCD**

    ```bash
    kubectl create namespace argocd
    ```

2. **Install ArgoCD**

    ```bash
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

### Step 2: Access the ArgoCD API Server

1. **Forward the ArgoCD server port**

    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

2. **Get the initial admin password**

    ```bash
    kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name
    ```

   Use the name of the pod to get the password:

    ```bash
    kubectl -n argocd get secret <argocd-server-pod-name> -o jsonpath="{.data.password}" | base64 --decode
    echo
    ```

3. **Access the ArgoCD UI**

   Open your browser and go to `http://localhost:8080`. Login with the username `admin` and the password retrieved in the previous step.

4. **Create a new namespace for application**

   ```bash
    kubectl create namespace app-argocd
    ```

## Prometheus and Grafana using Helm
### Step 1: Add Helm Repositories

First, you need to add the Helm repositories for Prometheus and Grafana.

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Step 2: Install Prometheus and Grafana

To install the complete Prometheus and Grafana stack, use the `kube-prometheus-stack` chart from the Prometheus Community Helm repository:

```sh
helm install prometheus prometheus-community/kube-prometheus-stack
```

### Step 3: Accessing Grafana

To access the Grafana dashboard, you need to port forward the Grafana service.

```sh
kubectl port-forward service/prometheus-grafana 8081:80
```

Grafana will be accessible at `http://localhost:8081`.

Get the default admin password for Grafana:

```sh
kubectl get secret --namespace default prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
echo
```

The default username is `admin`.

# Build docker images and Deploy  to Azure 
## Push Docker Image to ACR
1. Login to your container registry
```bash 
docker login <acr_registry_name>.azurecr.io
```
2. Set Up Docker Buildx:
   If you haven't already, enable and set up Docker Buildx:
2. Push to your registry
```bash
docker tag <docker_image_name> <acr_registry_name>.azurecr.io/<docker_image_name>:<version>
docker push <acr_registry_name>.azurecr.io/<docker_image_name>:<version>
```
3. Verify docker image on ACR
```bash
az acr repository list --name <acr_registry_name> --output table
```
## Deploy AKS
### Config kubectl
```bash
az aks get-credentials --resource-group <resource_group_name> --name <aks_name> --overwrite-existing
```
### Deploy
1. Mongo
```bash
kubectl apply -f k8s/azure/mongodb.yaml
```
2. Backend
```bash
kubectl apply -f k8s/azure/backend.yaml
```
3. Frontend
```bash
kubectl apply -f k8s/azure/frontend.yaml
```