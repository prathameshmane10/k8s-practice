# Kubernetes Practical Lab: Multi-Node Kind Cluster & Dashboard on AWS EC2

A hands-on walkthrough of provisioning a multi-node Kubernetes cluster with **Kind** (Kubernetes in Docker) on an AWS EC2 instance, deploying the Kubernetes Dashboard, and exposing it securely for remote access.

## Table of Contents

1. [Overview](#1-overview)
2. [Prerequisites](#2-prerequisites)
3. [Kind Cluster Creation](#3-kind-cluster-creation)
4. [Kubernetes Dashboard Deployment](#4-kubernetes-dashboard-deployment)
5. [Dashboard Access & Port Forwarding](#5-dashboard-access--port-forwarding)
6. [Troubleshooting](#6-troubleshooting)
7. [Directory Structure](#7-directory-structure)

---

## 1. Overview

This project sets up a multi-node Kubernetes development environment without a managed service like EKS. An AWS EC2 Ubuntu instance acts as the host, running a Kind cluster of **1 control-plane node and 2 worker nodes** as Docker containers.

**What this covers:**

- Provisioning a multi-node Kubernetes cluster with Kind
- Managing cluster resources via `kubectl`
- Deploying the official Kubernetes Dashboard UI
- Creating an admin ServiceAccount and RBAC binding
- Exposing the Dashboard over an EC2 public network interface

---

## 2. Prerequisites

### EC2 Instance

| Setting | Value |
|---|---|
| OS | Ubuntu 22.04 LTS / 24.04 LTS |
| Instance type | `t2.medium` or higher (2 vCPU, 4 GB RAM minimum) |
| Storage | 20 GB GP3 EBS volume |

Multi-node Kind runs each node as a Docker container, so anything smaller than `t2.medium` will struggle.

### Host Tools

Install on the EC2 instance before creating the cluster:

- **Docker Engine** — container runtime for the Kind node containers
- **kubectl** — CLI for talking to the cluster API server
- **Kind** — runs local Kubernetes clusters using Docker "nodes"

An `install.sh` script in this repo automates these installations.

---

## 3. Kind Cluster Creation

Cluster topology is defined in `kind_config.yml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Create the cluster:

```bash
kind create cluster --name my-cluster --config kind_config.yml
```

Verify all three nodes are `Ready`:

```bash
kubectl get nodes
```

Expected: `my-cluster-control-plane`, `my-cluster-worker`, `my-cluster-worker2`.

---

## 4. Kubernetes Dashboard Deployment

### Step 1 — Deploy the Dashboard manifests

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

This creates the `kubernetes-dashboard` namespace along with the Dashboard UI and its metrics scraper.

### Step 2 — Create an admin ServiceAccount (`dashboard-admin.yml`)

The Dashboard ships with minimal permissions by default. This binds a dedicated ServiceAccount to the `cluster-admin` role:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```

Apply it:

```bash
kubectl apply -f dashboard-admin.yml
```

> **Note:** `cluster-admin` grants full cluster privileges. This is acceptable for a throwaway lab cluster, but scope the role down for anything shared or long-lived.

### Step 3 — Generate a login token

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Paste the resulting bearer token into the Dashboard login screen. Tokens are short-lived by default — re-run the command to get a fresh one.

---

## 5. Dashboard Access & Port Forwarding

The Dashboard service runs inside the cluster network. Forward it to the host's interfaces:

```bash
kubectl port-forward --address 0.0.0.0 -n kubernetes-dashboard svc/kubernetes-dashboard 8443:443
```

`--address 0.0.0.0` binds the forward to all interfaces on the EC2 host so it is reachable from outside the instance.

Then open:

```
https://<EC2-PUBLIC-IP>:8443
```

---

## 6. Troubleshooting

### `ERR_CONNECTION_TIMED_OUT`

**Cause:** The EC2 security group is blocking inbound traffic on port 8443.

**Fix:** Add a Custom TCP inbound rule to the instance's security group:

- **Port range:** `8443`
- **Source:** your own workstation IP (`<your-ip>/32`)

Avoid `0.0.0.0/0` — that exposes a cluster-admin Dashboard to the entire internet.

### `NET::ERR_CERT_AUTHORITY_INVALID`

**Cause:** The Dashboard serves a self-signed TLS certificate that no public CA recognises.

**Fix:** Expected for a lab setup. Click **Advanced → Proceed** in the browser to continue.

### Dashboard pods not reaching `Running`

Check pod status and events:

```bash
kubectl get pods -n kubernetes-dashboard
kubectl describe pod <pod-name> -n kubernetes-dashboard
```

On an undersized instance this is usually resource pressure — confirm the instance type meets the minimum above.

---

## 7. Directory Structure

```
k8s-practice/
├── kind-cluster/
│   ├── README.md             # Detailed documentation for Kind cluster setup
│   ├── kind_config.yml       # Multi-node Kind cluster configuration
│   ├── dashboard-admin.yml   # RBAC configuration for Dashboard admin access
│   └── install.sh            # Setup & automation script
└── (planned) minikube-cluster/
```

---

## Cleanup

Delete the cluster and reclaim resources when you're done:

```bash
kind delete cluster --name my-cluster
```
