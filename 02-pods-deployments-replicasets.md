# 02 — Core Objects: Cluster, Nodes, Pods, Deployments, ReplicaSets

## Kubernetes cluster (what it is)
A **Kubernetes cluster** is a group of servers (machines) working together to run your containers.

Each server in the cluster is called a **node**.

Nodes can be:
- physical (bare metal)
- virtual machines (more common)

## Master node vs Worker nodes (control plane vs workload)
In a cluster:

### Master node (Control Plane)
- “Brain” of the cluster
- manages worker nodes
- runs system components
- usually does **not** run your app containers

### Worker nodes
- where your actual application pods run

> Note: In Minikube (local learning cluster), you often get a single node that acts as both master + worker.

## Pod (smallest unit in Kubernetes)
In Docker world, a container is the “smallest unit”.  
In Kubernetes, the smallest unit is a **pod**.

A pod contains:
- **one or more containers**
- shared storage (volumes)
- shared networking (same IP)

### Most common pod setup
Usually: **1 pod = 1 container**

### When multiple containers in one pod?
Sometimes containers must live “tightly together” and share the same network/volume space.

### Important rule: One pod runs on one node
You cannot split a single pod across multiple servers.

## How Kubernetes talks to itself (high level)
There are system components on nodes.

### On every node
- **container runtime**: runs containers (Docker/CRI-O/containerd)
- **kubelet**: node agent, talks to API Server
- **kube-proxy**: networking rules on the node

### On master/control plane
- **API Server**: main communication point (everything talks through it)
- **scheduler**: decides which node should run which pod
- **controller manager**: keeps the cluster in the desired state
- **cloud controller manager**: integrates with cloud providers (load balancers, etc.)
- **etcd**: key-value store (cluster “database” storing state/config)
- **DNS service**: resolves service names to IPs inside the cluster

## kubectl (how YOU manage Kubernetes)
`kubectl` is the command-line tool for Kubernetes.

You can run `kubectl` from your laptop and it talks to the cluster via:
- REST API
- HTTPS
- API Server

## Creating a pod directly (imperative quick test)
You can create a single pod like this:

```bash
kubectl run nginx --image=nginx
kubectl get pods
kubectl describe pod nginx
```

This is similar to `docker run`, but it creates a pod with a container inside it.

### Key thing about pod IPs
Pods get internal IPs, but these are not meant to be “stable” or used from outside the cluster.

## Deployments (the normal way in real life)
Creating a single pod is not convenient for scaling.

A **Deployment** is the standard way to run apps:
- creates pods for you
- can scale replicas (more pods)
- supports rolling updates

Create deployment (imperative style):
```bash
kubectl create deployment nginx-deployment --image=nginx
kubectl get deploy
kubectl get pods
kubectl describe deployment nginx-deployment
```

## ReplicaSet (the “manager” behind a deployment)
Deployment creates a **ReplicaSet**.  
ReplicaSet ensures you have the desired number of pod copies (replicas).

That’s why pod names look like:
- `nginx-deployment-<replicaset-hash>-<pod-hash>`

## Scaling a deployment
Scale up to 5 pods:
```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods
```

Scale down to 3 pods:
```bash
kubectl scale deployment nginx-deployment --replicas=3
kubectl get pods
```

## Deleting resources
Delete a pod:
```bash
kubectl delete pod <pod-name>
```

Delete deployment:
```bash
kubectl delete deployment nginx-deployment
```

## Alias tip (quality-of-life)
You can shorten `kubectl` to `k` temporarily:
```bash
alias k="kubectl"
k get pods
```

This alias lasts only for the current terminal session.

## Tiny glossary (for this chapter)
- **Cluster**: group of nodes
- **Node**: server in the cluster
- **Pod**: smallest unit, holds containers
- **Deployment**: manages pods (scaling + updates)
- **ReplicaSet**: ensures correct number of pods
- **kubectl**: CLI tool to control Kubernetes
