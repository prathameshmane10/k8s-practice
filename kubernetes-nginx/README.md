
# Kubernetes Nginx Deployment and Workload Practice

## 1. Project Overview

This project demonstrates hands-on Kubernetes practice using Nginx and different Kubernetes resources.

The project covers Namespace, Pod, Deployment, ReplicaSet, DaemonSet, Job, CronJob, Persistent Volume (PV), Persistent Volume Claim (PVC), and Service. It also includes resource verification, troubleshooting, networking, storage, and cleanup commands.

The objective is to understand how Kubernetes manages application workloads, provides stable network access to Pods, maintains persistent storage, and executes scheduled tasks.

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
│       ├── cron-job.yml
│       ├── service.yml
│       ├── persistent-volume.yml
│       └── persistent-volume-claim.yml
└── minikube/
```

## 3. Prerequisites

Ensure that you have a running Kubernetes cluster, `kubectl` installed, and access to the cluster.

Check the current Kubernetes context:

```bash
kubectl config current-context
```

Check cluster information:

```bash
kubectl cluster-info
```

Check cluster nodes:

```bash
kubectl get nodes
```

Navigate to the project directory:

```bash
cd ~/k8s-practice/kubernetes-nginx
```

## 4. Kubernetes Namespace

### What Is a Namespace?

A Namespace provides logical separation and organization for Kubernetes resources within a cluster.

In this project, the namespace is named `nginx`.

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

### Check ReplicaSets and Pods

```bash
kubectl get replicasets -n nginx
kubectl get pods -n nginx -o wide
```

### View Nginx Logs

```bash
kubectl logs -n nginx -l app=nginx
```

### Access Nginx Using Port Forwarding

Run the following command and keep the terminal open:

```bash
kubectl port-forward -n nginx deployment/nginx-deployment 8080:80
```

Open another terminal:

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

**Important:** The ReplicaSet and Deployment manifests use the same `app: nginx` label. Avoid running these controllers together during isolated practice because their selectors can overlap. Practice them separately or use distinct labels and selectors.

## 8. Kubernetes DaemonSet

### What Is a DaemonSet?

A DaemonSet ensures that a Pod runs on each eligible node in the Kubernetes cluster.

DaemonSets are commonly used for node-level services such as log collectors, monitoring agents, and networking components.

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

### Delete the DaemonSet

```bash
kubectl delete -f nginx/daemonsets.yml
```

**Important:** The DaemonSet also uses `app: nginx`. Practice it separately from the Deployment and ReplicaSet to avoid confusion caused by shared labels.

## 9. Kubernetes Job

### What Is a Job?

A Job creates Pods to execute a task until the required number of successful completions is reached.

Jobs are useful for batch processing, one-time scripts, database migrations, and other finite tasks.

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

### Delete the Job

```bash
kubectl delete -f nginx/job.yml
```

## 10. Kubernetes CronJob

### What Is a CronJob?

A CronJob creates Jobs according to a specified schedule. It is useful for recurring tasks such as backups, reports, scheduled maintenance, and periodic data processing.

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

### Check Jobs and Pods

```bash
kubectl get jobs -n nginx
kubectl get pods -n nginx
```

### View CronJob Pod Logs

```bash
kubectl logs -n nginx <cronjob-pod-name>
```

Replace `<cronjob-pod-name>` with an actual Pod name.

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
- A meaningful backup requires files to exist in `/data`.
- The manifest uses `hostPath` volumes, which refer to directories on the Kubernetes node where the Pod runs.
- Because the schedule runs every minute, it can create Jobs frequently.

### Delete the CronJob

```bash
kubectl delete -f nginx/cron-job.yml
```

## 11. Kubernetes Persistent Volume (PV) and Persistent Volume Claim (PVC)

### What Is a Persistent Volume?

A Persistent Volume (PV) is a storage resource available within a Kubernetes cluster. It defines storage capacity, access modes, storage type, and reclaim policy.

### What Is a Persistent Volume Claim?

A Persistent Volume Claim (PVC) is a request for storage by a user or application. Kubernetes matches the claim with a suitable Persistent Volume based on the storage requirements.

### What Is hostPath Storage?

The `hostPath` volume type uses a directory or file from the node's filesystem as storage.

**Note:** `hostPath` is useful for local development and learning. It has limitations for production workloads because the data is tied to a particular node.

### Manifest Files

- `nginx/persistent-volume.yml`
- `nginx/persistent-volume-claim.yml`

### Step 1: Create the Persistent Volume

```bash
kubectl apply -f nginx/persistent-volume.yml
```

### Step 2: Create the Persistent Volume Claim

```bash
kubectl apply -f nginx/persistent-volume-claim.yml
```

### Step 3: Verify PV and PVC Binding

```bash
kubectl get pv
kubectl get pvc -n nginx
```

### Describe PV and PVC

```bash
kubectl describe pv local-pv
kubectl describe pvc local-pvc -n nginx
```

### Troubleshooting PV and PVC

```bash
kubectl describe pv local-pv
kubectl describe pvc local-pvc -n nginx
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

Check storage capacity, access modes, storage class compatibility, and the `hostPath` configuration if the claim remains in `Pending` state.

### Cleanup PV and PVC

Delete the PVC first:

```bash
kubectl delete -f nginx/persistent-volume-claim.yml
```

Delete the PV afterward:

```bash
kubectl delete -f nginx/persistent-volume.yml
```

### PV and PVC Key Takeaways

- A PV represents storage available to the cluster.
- A PVC requests storage for an application.
- Kubernetes binds a compatible PV to a PVC.
- PVs are cluster-scoped, while PVCs are namespace-scoped.
- Deleting a PV does not necessarily delete the underlying `hostPath` directory.

## 12. Kubernetes Service

### What Is a Kubernetes Service?

A Kubernetes Service provides a stable network endpoint for accessing a group of Pods.

Pods can be created, deleted, or replaced during application updates. Their IP addresses may change. A Service provides a consistent way to reach the matching Pods without depending on individual Pod IP addresses.

### What Is a ClusterIP Service?

`ClusterIP` is the default Kubernetes Service type. It exposes an application through an internal IP address that is reachable from within the cluster.

It is commonly used for internal communication between application components.

### Manifest File

`nginx/service.yml`

### Service Configuration

The current Service manifest contains the following configuration:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### Explanation of the Configuration

| Field | Description |
|---|---|
| `kind: Service` | Defines the Kubernetes resource type |
| `name: nginx-service` | Name of the Service |
| `namespace: nginx` | Namespace containing the Service |
| `selector: app: nginx` | Selects Pods with the matching label |
| `protocol: TCP` | Network protocol used by the Service |
| `port: 80` | Port exposed by the Service |
| `targetPort: 80` | Port on the selected Pods |
| `type: ClusterIP` | Exposes the Service internally within the cluster |

### Step 1: Create the Service

Make sure the `nginx` namespace exists and matching Nginx Pods are running.

```bash
kubectl apply -f nginx/service.yml
```

### Step 2: Verify the Service

```bash
kubectl get services -n nginx
```

You can also use the short resource name:

```bash
kubectl get svc -n nginx
```

### Step 3: Describe the Service

```bash
kubectl describe service nginx-service -n nginx
```

This displays the Service IP, ports, selector, and associated endpoint information.

### Step 4: Check Service Endpoints

```bash
kubectl get endpoints nginx-service -n nginx
```

On Kubernetes versions that support EndpointSlices, you can also run:

```bash
kubectl get endpointslices -n nginx
```

The Service should have endpoints when matching Pods are available and ready.

If there are no endpoints, verify that the Pods have the `app: nginx` label and are ready to receive traffic.

### Step 5: Test the Service Using Port Forwarding

Run the following command:

```bash
kubectl port-forward -n nginx service/nginx-service 8080:80
```

Keep this terminal open.

Open another terminal and execute:

```bash
curl http://localhost:8080
```

If the matching Nginx Pods are running and ready, you should receive the Nginx welcome page HTML.

**Note:** Port forwarding is a testing method. It does not make the ClusterIP Service publicly accessible.

### Step 6: Access the Service from Inside the Cluster

If a suitable test Pod is available, you can test the Service using its DNS name:

```bash
curl http://nginx-service.nginx.svc.cluster.local
```

Run this command from a container or Pod inside the cluster that has `curl` installed and network access to the Service.

The standard Service DNS name follows this format:

```text
<service-name>.<namespace>.svc.cluster.local
```

### Step 7: Troubleshoot the Service

Check the Service configuration:

```bash
kubectl get svc nginx-service -n nginx -o wide
kubectl describe svc nginx-service -n nginx
```

Check the selected Pods:

```bash
kubectl get pods -n nginx --show-labels
```

Check the endpoints:

```bash
kubectl get endpoints nginx-service -n nginx
kubectl get endpointslices -n nginx
```

Check recent namespace events:

```bash
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

Common issues include:

- **No endpoints:** No ready Pods match the Service selector.
- **Connection refused:** The target container may not be listening on the expected port.
- **Connection timeout:** Network connectivity or cluster networking may be preventing communication.
- **Incorrect selector:** The Service selector does not match the Pod labels.
- **Port mismatch:** The Service `targetPort` does not match the application's listening port.

### Step 8: Delete the Service

```bash
kubectl delete -f nginx/service.yml
```

Verify that it has been removed:

```bash
kubectl get svc -n nginx
```

### Service Key Takeaways

- A Service provides a stable endpoint for accessing Pods.
- `ClusterIP` provides internal cluster access.
- A Service uses label selectors to identify matching Pods.
- `port` is the Service port, while `targetPort` is the destination port on the Pods.
- Kubernetes Service DNS allows applications to communicate using service names.
- A Service does not create application Pods; it routes traffic to matching Pods.

## 13. General Verification Commands

### View Common Resources

```bash
kubectl get all -n nginx
```

Note that `kubectl get all` does not include every Kubernetes resource type, including PVs and PVCs.

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

### View Jobs and CronJobs

```bash
kubectl get jobs -n nginx
kubectl get cronjobs -n nginx
```

### View Services

```bash
kubectl get services -n nginx
```

### View Persistent Volumes and Claims

```bash
kubectl get pv
kubectl get pvc -n nginx
```

### View Namespace Events

```bash
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

### Describe Resources

```bash
kubectl describe pod -n nginx <pod-name>
kubectl describe deployment nginx-deployment -n nginx
kubectl describe service nginx-service -n nginx
kubectl describe pv local-pv
kubectl describe pvc local-pvc -n nginx
```

Replace placeholders with actual resource names where required.

## 14. Troubleshooting

If a Pod or workload is not behaving as expected, inspect its status and events.

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
- PVC `Pending` — the claim has not been bound to a suitable volume.
- Service without endpoints — no ready Pods match the Service selector.

Always inspect the actual resource status and events before changing the manifest.

## 15. Cleanup

Delete resources after completing the exercises.

**Recommended order:** Delete the Service and workload resources first, then delete the PVC, PV, and finally the namespace if you no longer need it.

### Delete the Service

```bash
kubectl delete -f nginx/service.yml
```

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

### Delete the Persistent Volume Claim

```bash
kubectl delete -f nginx/persistent-volume-claim.yml
```

### Delete the Persistent Volume

```bash
kubectl delete -f nginx/persistent-volume.yml
```

### Verify Remaining Resources

```bash
kubectl get all -n nginx
kubectl get pvc -n nginx
kubectl get pv
```

### Delete the Namespace

Only delete the namespace if you no longer need its resources:

```bash
kubectl delete -f nginx/namespace.yml
```

Deleting the namespace removes its namespaced resources. Persistent Volumes are cluster-scoped and are not automatically removed simply because the namespace is deleted.

**Important:** Check the PV reclaim policy and underlying storage before removing data.

## 16. Useful Kubernetes Commands Reference

| Command | Purpose |
|---|---|
| `kubectl get nodes` | List cluster nodes |
| `kubectl get namespaces` | List namespaces |
| `kubectl get pods -n nginx` | List Pods |
| `kubectl get deployments -n nginx` | List Deployments |
| `kubectl get replicasets -n nginx` | List ReplicaSets |
| `kubectl get daemonsets -n nginx` | List DaemonSets |
| `kubectl get jobs -n nginx` | List Jobs |
| `kubectl get cronjobs -n nginx` | List CronJobs |
| `kubectl get svc -n nginx` | List Services |
| `kubectl get pv` | List Persistent Volumes |
| `kubectl get pvc -n nginx` | List Persistent Volume Claims |
| `kubectl describe svc nginx-service -n nginx` | Inspect Service configuration |
| `kubectl describe pv local-pv` | Inspect Persistent Volume details |
| `kubectl get events -n nginx` | View namespace events |
| `kubectl delete -f <file>` | Delete resources defined in a manifest |

## 17. Learning Outcomes

Through this practice, I learned how to:

- Organize Kubernetes resources using Namespaces.
- Run containers using Pods.
- Manage application replicas using Deployments and ReplicaSets.
- Run node-level workloads using DaemonSets.
- Execute one-time tasks using Jobs.
- Schedule recurring tasks using CronJobs.
- Configure persistent storage using PVs and PVCs.
- Expose applications internally using ClusterIP Services.
- Understand Service selectors, ports, endpoints, and DNS.
- Inspect resources and troubleshoot common Kubernetes issues.
- Clean up resources safely after completing practical exercises.

## Practice Status

This repository documents hands-on Kubernetes practice with workloads, storage, networking, verification, troubleshooting, and resource cleanup.
