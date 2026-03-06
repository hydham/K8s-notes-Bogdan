# 04 — Hands-on Flow: Minikube, YAML (Declarative), Rollouts, Dashboard, Runtime Switch

## Local cluster setup (Minikube)
To learn Kubernetes for free, you can run a local cluster using **Minikube**.

Minikube typically creates:
- a single node cluster
- node acts as master + worker (in local setup)

## Minikube needs a “driver”
Minikube runs the cluster inside:
- VirtualBox / VMware / Parallels / Hyper-V, etc.
- or Docker (not recommended in the lecture for runtime-switch demo)

Examples from lecture:
- Windows: Hyper-V recommended
- macOS: VirtualBox is free and OK

## Install tools

### Install kubectl
macOS (brew):
```bash
brew install kubectl
kubectl version --client
```

Windows: install via Chocolatey or Scoop (lecture recommends Chocolatey).

### Install Minikube
macOS (brew):
```bash
brew install minikube
minikube version
```

Windows: `choco install minikube`

## Start a cluster
Check status first:
```bash
minikube status
```

Start (example with VirtualBox):
```bash
minikube start --driver=virtualbox
```

After start:
```bash
minikube status
kubectl cluster-info
kubectl get nodes
```

## SSH into minikube node (optional learning)
Get node IP:
```bash
minikube ip
```

SSH:
```bash
ssh docker@<minikube-ip>
# password: tcuser
```

Inside node, you can see containers (if runtime is Docker):
```bash
docker ps
```

## Imperative vs Declarative

### Imperative (commands)
You tell Kubernetes “do it now”:
- `kubectl create deployment ...`
- `kubectl expose ...`
- `kubectl scale ...`

Fast for demos, but not ideal for real teams.

### Declarative (YAML files)
You write “desired state” in YAML, then apply it:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

This is the common real-world style.

## Deployment YAML (core pattern)
A Deployment YAML usually includes:
- `apiVersion: apps/v1`
- `kind: Deployment`
- `metadata.name`
- `spec.replicas`
- `spec.selector.matchLabels`
- `spec.template.metadata.labels`
- `spec.template.spec.containers` (name, image, ports, resources)

Scaling in YAML is just:
```yaml
spec:
  replicas: 5
```

Re-apply after editing:
```bash
kubectl apply -f deployment.yaml
```

## Service YAML (core pattern)
Service YAML includes:
- `apiVersion: v1`
- `kind: Service`
- `metadata.name`
- `spec.selector`
- `spec.ports`
- `spec.type` (ClusterIP / NodePort / LoadBalancer)

Example port mapping idea:
- service port can be different from container port
- use `targetPort` if needed

## One YAML file can contain multiple resources
You can put multiple objects in one file using:
```yaml
---
```

Example:
- Service spec
---
- Deployment spec

## Rolling updates (smooth version upgrades)
Kubernetes deployment strategy in the lecture uses **RollingUpdate**.

Meaning:
- it creates new pods using the new image
- while old pods are still running
- then replaces them gradually
- minimizes downtime

Update image:
```bash
kubectl set image deployment/k8s-web-hello k8s-web-hello=<image>:<tag>
```

Watch rollout:
```bash
kubectl rollout status deployment/k8s-web-hello
```

List pods after rollout:
```bash
kubectl get pods
```

You’ll notice pods are “new” (age resets).

## Self-healing demo (delete a pod)
If deployment expects 4 replicas and you delete 1 pod:
- Kubernetes will create a new one automatically

```bash
kubectl delete pod <pod-name>
kubectl get pods
```

## Kubernetes Dashboard (Minikube)
Start dashboard:
```bash
minikube dashboard
```

You can visually inspect:
- deployments
- replica sets
- pods
- services
- nodes
- events

Stop dashboard proxy (Ctrl+C in terminal).

## Delete everything quickly (cleanup)
Delete resources created in default namespace:
```bash
kubectl delete all --all
```

Or delete by files:
```bash
kubectl delete -f deployment.yaml -f service.yaml
```

## Switching container runtime to CRI-O (important concept)
Lecture shows Kubernetes doesn’t depend on Docker.

Steps shown:

1) Stop and delete current minikube cluster:
```bash
minikube stop
minikube delete
```

2) Start again with CRI-O runtime:
```bash
minikube start --driver=virtualbox --container-runtime=cri-o
```

3) Verify Docker isn’t running inside node:
```bash
ssh docker@<minikube-ip>
docker ps   # should fail
```

4) Use `crictl` to see running containers (CRI runtime tool):
```bash
sudo crictl ps
```

Then you can deploy apps again, and it will work under CRI-O.

## Mini glossary (for this chapter)
- **Minikube**: local K8s cluster for learning
- **Driver**: virtualization/container backend Minikube uses
- **Imperative**: “run this command now”
- **Declarative**: “apply this desired state YAML”
- **Rollout**: updating pods gradually
- **CRI-O**: container runtime alternative to Docker
- **crictl**: CLI to inspect CRI runtime containers
