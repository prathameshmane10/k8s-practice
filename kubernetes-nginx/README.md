# Kubernetes Nginx Deployment and Workload Practice

## 1. Project Overview

This project demonstrates hands-on Kubernetes practice using Nginx and different Kubernetes workload resources.

The project covers Namespace, Pod, Deployment, ReplicaSet, DaemonSet, Job, and CronJob. It also includes resource verification, troubleshooting, port forwarding, and cleanup commands.

The purpose of this project is to understand how Kubernetes manages application workloads, maintains the desired number of replicas, executes batch tasks, and schedules recurring jobs.

## 2. Project Directory Structure

```text
k8s-practice/
├── kind-cluster/
├── kubeadm/
├── kubernetes-nginx/
│   ├── README.md
│   └── nginx/
│       ├── namespace.yml
│       ├── pod.yml
│       ├── deployment.yml
│       ├── replicasets.yml
│       ├── daemonsets.yml
│       ├── job.yml
│       └── cron-job.yml
└── minikube/
```

## 3. Prerequisites

Before starting, ensure that you have a running Kubernetes cluster, kubectl installed and configured, and access to the cluster.

Check the current Kubernetes context:

```bash
kubectl config current-context
```

Check the cluster connection:

```bash
kubectl cluster-info
```

Check available cluster nodes:

```bash
kubectl get nodes
```

Navigate to the project directory:

```bash
cd ~/k8s-practice/kubernetes-nginx
```

## 4. Kubernetes Namespace

### What Is a Namespace?

A Namespace is a logical way to organize and separate Kubernetes resources within a cluster. In this project, the namespace is named `nginx`.

### Manifest File

`nginx/namespace.yml`

### Create the Namespace

```bash
kubectl apply -f nginx/namespace.yml
```

### Verify the Namespace

```bash
kubectl get namespaces
kubectl get namespace nginx
```

## 5. Kubernetes Pod

### What Is a Pod?

A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that share networking and can share storage.

In this project, a standalone Pod runs an Nginx container.

### Manifest File

`nginx/pod.yml`

### Create the Pod

```bash
kubectl apply -f nginx/pod.yml
```

### Verify the Pod

```bash
kubectl get pods -n nginx
kubectl get pods -n nginx -o wide
```

### Describe the Pod

```bash
kubectl describe pod -n nginx <pod-name>
```

Replace `<pod-name>` with the actual Pod name.

### View Pod Logs

```bash
kubectl logs -n nginx <pod-name>
```

### Delete the Pod

```bash
kubectl delete -f nginx/pod.yml
```

A standalone Pod is not automatically recreated if it is deleted. Controllers such as Deployments and ReplicaSets help maintain the desired number of Pods.

## 6. Kubernetes Deployment

### What Is a Deployment?

A Deployment manages application releases and ReplicaSets. It helps maintain the desired number of replicas and supports rolling updates and rollbacks.

The Nginx Deployment in this project is configured to run two replicas.

### Manifest File

`nginx/deployment.yml`

### Create the Deployment

```bash
kubectl apply -f nginx/deployment.yml
```

### Verify the Deployment

```bash
kubectl get deployments -n nginx
kubectl get deployment nginx-deployment -n nginx
```

### Describe the Deployment

```bash
kubectl describe deployment nginx-deployment -n nginx
```

### Check ReplicaSets

```bash
kubectl get replicasets -n nginx
```

### Check Pods

```bash
kubectl get pods -n nginx -o wide
```

### View Nginx Logs

```bash
kubectl logs -n nginx -l app=nginx
```

### Access Nginx Using Port Forwarding

Run this command and keep the terminal open:

```bash
kubectl port-forward -n nginx deployment/nginx-deployment 8080:80
```

Open another terminal and run:

```bash
curl http://localhost:8080
```

If the Deployment is running correctly, you should receive the Nginx welcome page HTML.

### Delete the Deployment

```bash
kubectl delete -f nginx/deployment.yml
```

## 7. Kubernetes ReplicaSet

### What Is a ReplicaSet?

A ReplicaSet ensures that a specified number of matching Pod replicas are running at a given time.

For example, if a ReplicaSet specifies two replicas, Kubernetes attempts to maintain two matching Pods.

In most application deployments, ReplicaSets are managed through Deployments instead of being created directly.

### Manifest File

`nginx/replicasets.yml`

### Create the ReplicaSet

```bash
kubectl apply -f nginx/replicasets.yml
```

### Verify the ReplicaSet

```bash
kubectl get replicasets -n nginx
```

### Describe the ReplicaSet

```bash
kubectl describe replicaset nginx-replicasets -n nginx
```

### Check Pods

```bash
kubectl get pods -n nginx -l app=nginx -o wide
```

### Delete the ReplicaSet

```bash
kubectl delete -f nginx/replicasets.yml
```

**Important:** The ReplicaSet and Deployment manifests use the same `app: nginx` label. Avoid running these controllers together during isolated practice because their selectors can overlap and make resource behavior confusing. Practice them separately or use distinct labels and selectors.

## 8. Kubernetes DaemonSet

### What Is a DaemonSet?

A DaemonSet ensures that a Pod runs on each eligible node in the Kubernetes cluster.

DaemonSets are commonly used for node-level services such as log collectors, monitoring agents, and networking components.

In this project, the DaemonSet runs an Nginx container.

### Manifest File

`nginx/daemonsets.yml`

### Create the DaemonSet

```bash
kubectl apply -f nginx/daemonsets.yml
```

### Verify the DaemonSet

```bash
kubectl get daemonsets -n nginx
```

### Describe the DaemonSet

```bash
kubectl describe daemonset nginx-daemonsets -n nginx
```

### Check Pods and Nodes

```bash
kubectl get pods -n nginx -o wide
kubectl get nodes
```

The number of DaemonSet Pods depends on the number of eligible nodes and any scheduling restrictions.

### Delete the DaemonSet

```bash
kubectl delete -f nginx/daemonsets.yml
```

**Important:** The DaemonSet also uses `app: nginx`. Practice it separately from the Deployment and ReplicaSet to avoid confusion caused by shared labels.

## 9. Kubernetes Job

### What Is a Job?

A Job creates Pods to execute a task until the required number of successful completions is reached.

Jobs are useful for batch processing, one-time scripts, database migrations, and other finite tasks.

In this project, a BusyBox container prints a message and sleeps for 10 seconds.

### Manifest File

`nginx/job.yml`

### Create the Job

```bash
kubectl apply -f nginx/job.yml
```

### Verify the Job

```bash
kubectl get jobs -n nginx
```

### Describe the Job

```bash
kubectl describe job demo-job -n nginx
```

### Check Job Pods

```bash
kubectl get pods -n nginx -l app=batch-task
```

### View Job Logs

```bash
kubectl logs -n nginx -l app=batch-task
```

### Important Job Configuration

The current manifest uses:

- `completions: 1` — one successful completion is required.
- `parallelism: 2` — Kubernetes may run up to two Pods concurrently for the Job.
- `restartPolicy: Never` — a failed container is not restarted inside the same Pod; the Job controller may create another Pod depending on its settings.

Because the task is short, the Job may complete quickly.

### Delete the Job

```bash
kubectl delete -f nginx/job.yml
```

## 10. Kubernetes CronJob

### What Is a CronJob?

A CronJob creates Jobs according to a specified schedule. It is useful for recurring tasks such as backups, reports, scheduled maintenance, and periodic data processing.

In this project, a BusyBox container runs on a one-minute schedule and executes directory creation and copy commands.

### Manifest File

`nginx/cron-job.yml`

### Create the CronJob

```bash
kubectl apply -f nginx/cron-job.yml
```

### Verify the CronJob

```bash
kubectl get cronjobs -n nginx
```

### Describe the CronJob

```bash
kubectl describe cronjob minute-backup -n nginx
```

### Check Jobs Created by the CronJob

```bash
kubectl get jobs -n nginx
```

### Check Pods Created by the CronJob

```bash
kubectl get pods -n nginx
```

### View CronJob Pod Logs

```bash
kubectl logs -n nginx <cronjob-pod-name>
```

Replace `<cronjob-pod-name>` with an actual Pod name created by the CronJob.

### CronJob Schedule Explanation

The current schedule is:

```yaml
schedule: "* * * * *"
```

This means the CronJob is scheduled to run once every minute.

The five cron fields represent:

```text
Minute Hour Day-of-Month Month Day-of-Week
```

### Important CronJob Notes

- The current command creates `/data` and `/backups` directories and copies `/data` into `/backups`.
- A meaningful backup requires files to exist in `/data`; otherwise, there may be little or no data to copy.
- The manifest uses `hostPath` volumes, which refer to directories on the Kubernetes node where the Pod runs.
- `hostPath` storage can be node-specific and should be used carefully outside a learning environment.
- The shell command uses semicolons and `&&` together, so the final completion message might still print even if an earlier command fails.
- Because the schedule runs every minute, it can create Jobs frequently.

### Delete the CronJob

```bash
kubectl delete -f nginx/cron-job.yml
```

Delete the CronJob when you have finished practicing so that it stops scheduling new Jobs.

## 11. Apply All Manifests

Create the namespace first:

```bash
kubectl apply -f nginx/namespace.yml
```

You can then apply individual workload manifests for practice.

### Apply the Deployment

```bash
kubectl apply -f nginx/deployment.yml
```

### Apply the Standalone Pod

```bash
kubectl apply -f nginx/pod.yml
```

### Apply the ReplicaSet

```bash
kubectl apply -f nginx/replicasets.yml
```

### Apply the DaemonSet

```bash
kubectl apply -f nginx/daemonsets.yml
```

### Apply the Job

```bash
kubectl apply -f nginx/job.yml
```

### Apply the CronJob

```bash
kubectl apply -f nginx/cron-job.yml
```

**Recommended practice:** Apply, inspect, and delete each workload individually. Avoid running the Deployment, ReplicaSet, and DaemonSet together with their current shared Nginx labels.

## 12. General Verification Commands

### View All Common Resources

```bash
kubectl get all -n nginx
```

### View Pods

```bash
kubectl get pods -n nginx
kubectl get pods -n nginx -o wide
```

### View Deployments

```bash
kubectl get deployments -n nginx
```

### View ReplicaSets

```bash
kubectl get replicasets -n nginx
```

### View DaemonSets

```bash
kubectl get daemonsets -n nginx
```

### View Jobs

```bash
kubectl get jobs -n nginx
```

### View CronJobs

```bash
kubectl get cronjobs -n nginx
```

### View Namespace Events

```bash
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

### Describe a Resource

```bash
kubectl describe pod -n nginx <pod-name>
kubectl describe deployment nginx-deployment -n nginx
kubectl describe replicaset nginx-replicasets -n nginx
kubectl describe daemonset nginx-daemonsets -n nginx
kubectl describe job demo-job -n nginx
kubectl describe cronjob minute-backup -n nginx
```

Replace placeholders with actual resource names where required.

## 13. Troubleshooting

If a Pod is not starting or a workload is not behaving as expected, inspect its status and events.

### Check Pod Status

```bash
kubectl get pods -n nginx -o wide
```

### Inspect Pod Details

```bash
kubectl describe pod -n nginx <pod-name>
```

### View Container Logs

```bash
kubectl logs -n nginx <pod-name>
```

### Check Recent Events

```bash
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

Common errors include:

- `ImagePullBackOff` — Kubernetes cannot pull the container image.
- `ErrImagePull` — the image pull failed.
- `CrashLoopBackOff` — the container repeatedly exits and restarts.
- `Pending` — the Pod has not been scheduled or cannot start yet.
- Failed scheduling events — the cluster cannot place the Pod on a suitable node.

Always inspect the actual resource status and events before changing the manifest.

## 14. Cleanup

Delete resources after completing the exercises.

### Delete the CronJob

```bash
kubectl delete -f nginx/cron-job.yml
```

### Delete the Job

```bash
kubectl delete -f nginx/job.yml
```

### Delete the DaemonSet

```bash
kubectl delete -f nginx/daemonsets.yml
```

### Delete the ReplicaSet

```bash
kubectl delete -f nginx/replicasets.yml
```

### Delete the Standalone Pod

```bash
kubectl delete -f nginx/pod.yml
```

### Delete the Deployment

```bash
kubectl delete -f nginx/deployment.yml
```

### Verify Remaining Resources

```bash
kubectl get all -n nginx
kubectl get cronjobs,jobs -n nginx
```

### Delete the Namespace

```bash
kubectl delete -f nginx/namespace.yml
```

Deleting the namespace removes the namespaced resources inside it. Do not delete it if you still need other practice resources in the same namespace.




# Kubernetes Persistent Volume (PV) and Persistent Volume Claim (PVC) Practice

## Overview

This practice covers Kubernetes Persistent Volumes (PV) and Persistent Volume Claims (PVC) using a local `hostPath` storage configuration.

The resources are configured to work with the `nginx` namespace. This exercise demonstrates how Kubernetes storage resources are created, managed, inspected, and deleted.

## Objectives

- Understand Persistent Volumes (PV) and Persistent Volume Claims (PVC).
- Create a Persistent Volume using `hostPath` storage.
- Create a Persistent Volume Claim to request storage.
- Verify the binding between PV and PVC.
- Inspect storage resources using `kubectl`.
- Practice cleaning up Kubernetes storage resources.

## Concepts Covered

### 1. Persistent Volume (PV)

A Persistent Volume is a storage resource available within a Kubernetes cluster. It defines the storage capacity, access modes, storage type, and reclaim policy.

### 2. Persistent Volume Claim (PVC)

A Persistent Volume Claim is a request for storage by a user or application. Kubernetes matches the claim with a suitable Persistent Volume based on the requested capacity, access modes, and storage requirements.

### 3. hostPath Storage

The `hostPath` volume type uses a directory or file from the node's filesystem as storage.

**Note:** `hostPath` is suitable for local development and learning purposes. It is generally not recommended for production workloads because the data is tied to a particular node.

## Project Structure

```text
k8s-practice/
└── nginx/
    ├── persistent-volume.yml
    └── persistent-volume-claim.yml
```

## Hands-On Practice

### Step 1: Create the Persistent Volume

Apply the Persistent Volume manifest.

```bash
kubectl apply -f nginx/persistent-volume.yml
```

Verify the Persistent Volume:

```bash
kubectl get pv
```

Inspect its configuration and status:

```bash
kubectl describe pv local-pv
```

### Step 2: Create the Persistent Volume Claim

Apply the Persistent Volume Claim manifest.

```bash
kubectl apply -f nginx/persistent-volume-claim.yml
```

Verify the claim in the `nginx` namespace:

```bash
kubectl get pvc -n nginx
```

Inspect the claim details:

```bash
kubectl describe pvc local-pvc -n nginx
```

### Step 3: Verify PV and PVC Binding

Check the Persistent Volume status:

```bash
kubectl get pv
```

Check the Persistent Volume Claim status:

```bash
kubectl get pvc -n nginx
```

Display Persistent Volumes and claims together:

```bash
kubectl get pv,pvc -n nginx
```

**Expected result:**

- The Persistent Volume should show `Bound` when successfully claimed.
- The Persistent Volume Claim should show `Bound` when successfully matched with a Persistent Volume.
- The PV and PVC should reference each other through their binding information.

Note: PVs are cluster-scoped resources, while PVCs are namespace-scoped resources. If the combined command does not display both resource types as expected, use the separate commands above.

### Step 4: Troubleshooting

If the PV or PVC remains in `Pending` state, inspect the resource details:

```bash
kubectl describe pv local-pv
kubectl describe pvc local-pvc -n nginx
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

Check the following:

- The requested storage capacity matches the available PV capacity.
- The access modes are compatible.
- The PV and PVC storage classes are compatible.
- The PVC references the correct namespace and volume configuration.
- The `hostPath` directory exists on the appropriate Kubernetes node, if required by the manifest.

### Step 5: Cleanup

Delete the Persistent Volume Claim first:

```bash
kubectl delete -f nginx/persistent-volume-claim.yml
```

Delete the Persistent Volume afterward:

```bash
kubectl delete -f nginx/persistent-volume.yml
```

Verify that the resources have been removed:

```bash
kubectl get pv
kubectl get pvc -n nginx
```

**Important:** Deleting a PV does not necessarily delete the underlying `hostPath` directory. Check the reclaim policy and the actual node filesystem before removing stored data.

## Key Takeaways

- A PV represents storage available to the Kubernetes cluster.
- A PVC requests storage for an application.
- Kubernetes binds a compatible PV to a PVC.
- PVs are cluster-scoped, whereas PVCs belong to namespaces.
- `kubectl describe` helps investigate resource configuration and events.
- `hostPath` is useful for learning but has limitations for production environments.

## Useful Commands Reference

| Command | Purpose |
|---|---|
| `kubectl apply -f <file>` | Create or update resources |
| `kubectl get pv` | List Persistent Volumes |
| `kubectl get pvc -n nginx` | List PVCs in the nginx namespace |
| `kubectl describe pv local-pv` | Inspect PV details |
| `kubectl describe pvc local-pvc -n nginx` | Inspect PVC details |
| `kubectl get events -n nginx` | View namespace events |
| `kubectl delete -f <file>` | Delete resources defined in a manifest |

## Practice Status

This section documents hands-on practice with Kubernetes Persistent Volumes, Persistent Volume Claims, storage binding, resource inspection, troubleshooting, and cleanup.


