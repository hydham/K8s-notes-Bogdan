# Chapter 4 — Pods, Deployments, Scaling, and Services (nginx example)

We will start with a simple image: **nginx** from Docker Hub.

## Create a pod directly (imperative style)

This is like `docker run`, but for Kubernetes pods:

```bash
kubectl run nginx --image=nginx
kubectl get pods
```

At first you may see `ContainerCreating`. After a moment you should see `Running`.

### Inspect the pod
```bash
kubectl describe pod nginx
```

You will see:
- namespace (default)
- node where it runs
- pod IP (internal)
- events: pulled image, created container, started container

Important: That pod IP is internal to the cluster networking. You generally cannot connect to it from your laptop.

### What’s happening inside the node (pause container)
If you SSH into the node and check containers, you’ll see two containers related to the pod:
- the actual nginx container
- a “pause” container

The pause container exists to hold the pod’s namespaces (network), so the pod identity stays stable while containers inside can be restarted.

(You can observe it with `docker ps | grep nginx` inside the Minikube node.)

## Delete the pod
```bash
kubectl delete pod nginx
kubectl get pods
```

Now we switch to the real-world approach: **deployments**.

## Create a deployment (imperative style)

```bash
kubectl create deployment nginx-deployment --image=nginx
kubectl get deployments
kubectl get pods
```

### Describe deployment
```bash
kubectl describe deployment nginx-deployment
```

You will notice:
- a selector label like `app=nginx-deployment`
- a ReplicaSet created under the hood
- events about scaling up the ReplicaSet

### ReplicaSet and pod names
Pods will look like:
`nginx-deployment-<replicaset-hash>-<pod-hash>`

That prefix is the ReplicaSet identity. The final part is unique per pod.

## Scale the deployment

Scale up to 5:
```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods
```

Scale down to 3:
```bash
kubectl scale deployment nginx-deployment --replicas=3
kubectl get pods
```

In a multi-node cluster, Kubernetes would spread these pods across nodes. In Minikube (single node), all will be on the same node.

## Why you still can’t “connect” using pod IP
Even if you list pod IPs:
```bash
kubectl get pods -o wide
```
those IPs are internal and pods are replaceable. You should not depend on them.

To connect reliably, you create a **service**.

## Create a ClusterIP service for the deployment

We expose port 80 of nginx, but we can choose a different service port like 8080:

```bash
kubectl expose deployment nginx-deployment --port=8080 --target-port=80
kubectl get services
```

You will see:
- a service named `nginx-deployment`
- type `ClusterIP`
- a cluster IP (like `10.x.x.x`)
- port mapping `8080 -> 80`

### Test ClusterIP from inside the cluster
ClusterIP is internal-only. It usually won’t work from your laptop. But it will work from inside the node.

SSH into the Minikube node:
```bash
ssh docker@<MINIKUBE_IP>
# password: tcuser
```

Then:
```bash
curl http://<CLUSTER_IP>:8080
```

You should see the nginx welcome page.

### Describe the service
```bash
kubectl describe service nginx-deployment
```

You’ll see “Endpoints” listing the pods that are behind this service. Kubernetes load-balances across them.

## Clean up nginx example

Delete deployment and service:
```bash
kubectl delete deployment nginx-deployment
kubectl delete service nginx-deployment
kubectl get deploy
kubectl get svc
```

Now we move to the “real workflow”: build your own app → build image → push Docker Hub → deploy in Kubernetes.
