# Kubernetes Nginx Deployment Practice

## Project Overview

This project demonstrates how to create a Kubernetes namespace and deploy Nginx using a standalone Pod and a Deployment.

## Directory Structure

```text
kubernetes-nginx/
├── README.md
└── nginx/
    ├── namespace.yml
    ├── pod.yml
    └── deployment.yml
```

## Manifest Files

- **namespace.yml** – Creates a namespace named `nginx`.
- **pod.yml** – Creates a standalone Nginx Pod.
- **deployment.yml** – Creates an Nginx Deployment with 2 replicas.

## Prerequisites

- Kubernetes cluster
- kubectl installed and configured

## Deployment Steps

### 1. Create Namespace

```bash
kubectl apply -f nginx/namespace.yml
```

### 2. Deploy Nginx Using Deployment

```bash
kubectl apply -f nginx/deployment.yml
```

### 3. Deploy Standalone Pod (Optional)

```bash
kubectl apply -f nginx/pod.yml
```

**Note:** Use either the Deployment or standalone Pod for separate practice. Applying both creates additional Pods.

## Verification Commands

Check all resources:

```bash
kubectl get all -n nginx
```

Check Pods:

```bash
kubectl get pods -n nginx -o wide
```

Check Deployment:

```bash
kubectl describe deployment nginx-deployment -n nginx
```

View Nginx logs:

```bash
kubectl logs -n nginx -l app=nginx
```

Test Nginx using port forwarding:

```bash
kubectl port-forward -n nginx deployment/nginx-deployment 8080:80
```

Open another terminal and run:

```bash
curl http://localhost:8080
```

## Cleanup

Delete the Deployment:

```bash
kubectl delete -f nginx/deployment.yml
```

Delete the standalone Pod if created:

```bash
kubectl delete -f nginx/pod.yml
```

Delete the namespace:

```bash
kubectl delete -f nginx/namespace.yml
```

**Note:** Deleting the namespace also deletes the resources inside it.
