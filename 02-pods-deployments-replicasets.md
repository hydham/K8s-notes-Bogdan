# Chapter 2 — Core Concepts and Architecture (Pod, Node, Cluster, Control Plane)

Before we touch commands, you need to understand the “units” Kubernetes works with.

## Pod (the smallest unit in Kubernetes)

In Docker, the smallest unit you usually think about is a **container**.

In Kubernetes, the smallest unit is a **pod**.

A pod is a wrapper around one or more containers. Containers are created **inside** the pod.

A pod includes:
- one or more containers,
- shared volumes (optional),
- shared networking (one shared IP address for the pod).

Because the network is shared, all containers inside the same pod share the same IP and can talk over `localhost`. That is why multi‑container pods exist: sometimes containers are tightly coupled and must live together.

Most common case: **one container per pod**.

Important rule: **one pod runs on one server** (one node). You cannot split one pod across multiple nodes.

## Kubernetes cluster

A Kubernetes cluster consists of **nodes**.

A node is simply a server:
- bare metal (physical), or
- virtual machine (most common).

Pods run on nodes. Kubernetes schedules pods onto nodes automatically after you create the cluster.

But nodes do not magically form a cluster on their own. You create/configure the cluster first. After the initial configuration, Kubernetes takes over and deploys pods automatically.

## Master node (control plane) and worker nodes

In a Kubernetes cluster:
- One node is the **master node** (also called the **control plane**).
- Other nodes are **worker nodes**.

The master node manages worker nodes. Your application pods typically run on worker nodes. The master node runs system pods that keep the cluster working.

## Which services run where (high-level)

### On every node
- **container runtime** (Docker / CRI‑O / containerd) — runs the containers
- **kubelet** — talks to the master API Server and manages pods on that node
- **kube-proxy** — handles networking rules and communication inside/between nodes

### On the master node (control plane)
- **API Server** — central point of communication (everything talks to it)
- **scheduler** — decides where pods should run
- **controller manager** — “controls everything,” keeps cluster in desired state
- **cloud controller manager** — integrates with cloud provider features (like load balancers)
- **etcd** — key-value store of cluster state/configuration
- **DNS service** — name resolution inside the cluster

## How you control the cluster: kubectl

You manage Kubernetes using **kubectl** (kube-control).

kubectl can run on your local machine and talk to the master API Server over HTTPS using REST API.

Worker nodes communicate with the master in the same way: through the API Server.

This means you can manage a remote cluster from your laptop.

Now we’ll build a local cluster and start practicing with real commands.