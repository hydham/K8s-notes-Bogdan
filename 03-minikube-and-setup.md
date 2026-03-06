# Chapter 3 — Installing Tools and Creating a Local Cluster with Minikube

In production, Kubernetes often runs in the cloud (AWS, GCP, etc.). But for learning, we want a free local cluster. We’ll use **Minikube**.

Minikube creates a **single-node** Kubernetes cluster on your computer. That single node acts as both:
- control plane (master)
- worker node

That is enough for learning and testing.

## What you need to install

You need three things:
1) **Minikube** (creates the local cluster)
2) **kubectl** (controls the cluster)
3) A code editor (I use **VS Code**)

Minikube also needs a virtualization/container driver (choose one):
- VirtualBox
- VMware
- Docker
- Hyper‑V
- Parallels

Recommendations from the course:
- Windows: **Hyper‑V** (often available by default)
- macOS: **VirtualBox** (free) or Parallels/VMware
- Docker driver is possible, but I do **not** recommend it here because of runtime-switch limitations later.

## Install kubectl

Go to Kubernetes docs:
- `kubernetes.io` → Documentation → Getting Started → Install Tools

### macOS (Homebrew)
```bash
brew install kubectl
kubectl version --client
```

### Windows (recommended: Chocolatey)
Install Chocolatey, then:
```powershell
choco install kubernetes-cli
kubectl version --client
```

(You can also use Scoop if you prefer.)

## Install Minikube

### macOS (Homebrew)
```bash
brew install minikube
minikube version
minikube help
```

### Windows (Chocolatey)
```powershell
choco install minikube
minikube version
```

## Start Minikube cluster

First check status:
```bash
minikube status
```

If you see “profile not found,” that’s normal for first run.

Now start the cluster. Always specify a driver:

### Example (macOS with VirtualBox)
```bash
minikube start --driver=virtualbox
```

### Example (Windows with Hyper‑V)
```bash
minikube start --driver=hyperv
```

After it finishes, you should see something like:
- preparing Kubernetes (by default on Docker runtime)
- generating certificates
- booting control plane
- done: kubectl configured to use the Minikube cluster

Verify status:
```bash
minikube status
```

Healthy looks like:
- host: running
- kubelet: running
- apiserver: running
- kubeconfig: configured

## Inspect the cluster with kubectl

Cluster info:
```bash
kubectl cluster-info
```

List nodes:
```bash
kubectl get nodes
```

Because Minikube is single-node, you’ll see one node with roles like control-plane/master.

## SSH into the Minikube node (optional but very educational)

Get node IP:
```bash
minikube ip
```

SSH:
```bash
ssh docker@<MINIKUBE_IP>
```

When prompted:
- accept fingerprint: type `yes`
- password is: `tcuser`

Now you are inside the node (the virtual machine). If the runtime is Docker, run:
```bash
docker ps
```

You will see containers like:
- kube-apiserver
- kube-scheduler
- etcd
- kube-proxy
- coredns
- storage-provisioner

These are the system components we discussed earlier.

Exit SSH:
```bash
exit
```

## Namespaces: why “kubectl get pods” shows nothing at first

If you run:
```bash
kubectl get pods
```
you may see:
- “No resources found in default namespace.”

List namespaces:
```bash
kubectl get namespaces
```

Common ones:
- default
- kube-system
- kube-public
- kube-node-lease

System pods live in `kube-system`. List them:
```bash
kubectl get pods --namespace=kube-system
```

Now we’re ready to create pods and deployments.
