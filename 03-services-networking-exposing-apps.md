# 03 — Services, Networking, and Exposing Apps

## Problem: Pod IPs are not for “real usage”
Pods can be:
- created
- deleted
- moved to another node

Their IP addresses can change. So you should not “hardcode” pod IPs.

## Solution: Services
A **Service** gives a stable way to reach pods.

A Service can:
- provide a stable IP (ClusterIP)
- provide a stable DNS name (like `nginx`)
- load-balance traffic across multiple pods

Services connect to pods using **labels/selectors**.

## Service types (from the lecture)

### 1) ClusterIP (internal only)
- Default service type
- reachable only from inside the cluster

Use case: internal database service, internal API, etc.

Create ClusterIP service (example exposes deployment):
```bash
kubectl expose deployment nginx-deployment --port=8080 --target-port=80
kubectl get svc
```

Explanation:
- `--port=8080` is the service port
- `--target-port=80` is the container port inside the pod

From your laptop, ClusterIP usually won’t work (especially in Minikube VM mode).
From inside the cluster/node, it works.

### 2) NodePort (reachable via node IP + port)
- Kubernetes opens a port on the node (like `32142`)
- you access via: `NODE_IP:NODE_PORT`

Create NodePort service:
```bash
kubectl expose deployment k8s-web-hello --type=NodePort --port=3000
kubectl get svc
```

In Minikube, easiest way to open in browser:
```bash
minikube service k8s-web-hello
```

Get only URL:
```bash
minikube service k8s-web-hello --url
```

### 3) LoadBalancer (cloud-friendly)
- Cloud provider gives an external load balancer IP
- In Minikube, it often behaves like NodePort (EXTERNAL-IP may show as Pending)

Create LoadBalancer service:
```bash
kubectl expose deployment k8s-web-hello --type=LoadBalancer --port=3000
kubectl get svc
```

## Load balancing across replicas (what you actually see)
If a deployment has multiple pods (replicas), a service will spread requests across them.

In the lecture, the app returns:
- `Hello from <hostname>`

So you can refresh the page and see different hostnames → means traffic is balancing to different pods.

## Connecting deployments together (service DNS)
Inside a cluster, services get DNS names.

Example idea:
- You create an `nginx` deployment + service named `nginx` (ClusterIP)
- Another web deployment can call `http://nginx` and reach it

This is why naming the service matters:
- service name is stable
- ClusterIP can change, but name stays the same

DNS check from inside a pod:
```bash
kubectl exec -it <pod-name> -- nslookup nginx
```

You should see the ClusterIP for the `nginx` service.

Quick test request from inside a pod:
```bash
kubectl exec -it <pod-name> -- wget -qO- http://nginx
```

## Tiny glossary (for this chapter)
- **Service**: stable endpoint for pods
- **Selector/Labels**: how service finds matching pods
- **ClusterIP**: internal-only service IP
- **NodePort**: node IP + open port access
- **LoadBalancer**: external LB (usually cloud)
- **DNS in K8s**: resolves service names to IPs
