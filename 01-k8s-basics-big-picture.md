# Chapter 1 — Welcome, What Kubernetes Is, and What We’ll Build

Welcome. In this course, we’ll learn Kubernetes from zero and then immediately do hands-on practice. Kubernetes is the de‑facto standard for deploying **containerized applications** into production. It is open source, so you can use it for free.

If you already know Docker basics (what an image is, what a container is, how to build an image), you’re ready. Docker familiarity is the only prerequisite I assume.

## What Kubernetes is (simple definition)

Kubernetes is a **container orchestration** system.

With Docker, you can run containers on one computer. But when you want to run **many containers** across **many servers** (physical or virtual), you quickly run into problems: where to place them, how to scale them, how to recover when one fails, and how to connect services together.

Kubernetes solves this by automating the work. You tell Kubernetes what you want, for example:

“I want 5 containers of this image running.”

Kubernetes then:
- deploys them across nodes (servers),
- balances load,
- scales up/down when needed,
- monitors health and replaces failed containers/pods automatically.

## Why it’s called K8s

“Kubernetes” starts with **K** and ends with **S**. Between them there are **8 letters**. So we shorten it as **K8s**.

## What Kubernetes takes care of

Kubernetes mainly helps with:

1) **Automatic deployment** of containerized apps across servers (usually virtual servers nowadays).

2) **Load distribution** across multiple servers and multiple replicas so resources aren’t underutilized or overloaded.

3) **Auto‑scaling** (increase/decrease the number of running containers/pods).

4) **Monitoring + health checks** and **self‑healing** (replace failed containers/pods).

## Container runtime (Docker is not the only option)

Kubernetes runs containers using a **container runtime** on each node. Docker is one runtime, but Kubernetes also supports:
- **CRI‑O**
- **containerd**

So Kubernetes is not bound to Docker. Later, we will switch the runtime from Docker to CRI‑O to prove it.

## Course plan (what we will do in this exact order)

1) Learn the terminology and key features: cluster, node, pod, and what Kubernetes does.
2) Build a small Kubernetes cluster locally on our computers with **Minikube**.
3) Create and scale deployments.
4) Build a custom Docker image, push it to Docker Hub, and deploy it in Kubernetes.
5) Create services and deployments using **YAML** files (declarative approach).
6) Connect multiple deployments together over the network (using Kubernetes DNS and service names).
7) Switch container runtime from Docker to CRI‑O.

That’s the roadmap. Now we move into the core building blocks: pods and clusters.