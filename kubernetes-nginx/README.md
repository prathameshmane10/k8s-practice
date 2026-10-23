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

