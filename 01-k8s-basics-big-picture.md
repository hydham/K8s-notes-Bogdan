# 01 — Big Picture and Kubernetes Basics (Beginner Notes)

## What Kubernetes is (in plain words)
Kubernetes (often written as **K8s**) is a tool that helps you **run containers in production** without you manually managing everything.

Docker can run containers on one machine. But when you need:
- many containers,
- on many servers,
- with scaling and auto-healing,

…doing it manually becomes messy. Kubernetes solves that by managing it for you.

## Why people use Kubernetes
Kubernetes helps with:

### 1) Automated deployment across servers
You tell Kubernetes: “Run my app with 5 copies.”  
Kubernetes decides where to run those copies (which server/node).

### 2) Load distribution
If you have multiple copies of your app running, Kubernetes can spread traffic across them.

### 3) Auto-scaling
You can scale up (more copies) or scale down (fewer copies).

### 4) Health checks + self-healing
If a container crashes, Kubernetes can replace it automatically.

## Kubernetes is open-source
Kubernetes is **free** and open-source. It’s a “de facto standard” for running containerized apps in production.

## Why it’s called K8s
“Kubernetes” has 10 letters.  
Between **K** and **S** there are **8** letters → **K8s**.

## Container runtimes (Docker is not the only option)
Kubernetes runs containers using a **container runtime** on each server (node). Common runtimes mentioned:
- Docker
- CRI-O
- containerd

Important idea: Kubernetes can run **without Docker**.

## Course plan (what the lecture covers)
This lecture goes through:
- terminology and architecture (cluster, nodes, pods)
- hands-on: create a local cluster (Minikube)
- create and scale deployments
- build a custom Docker image, push to Docker Hub, deploy it
- use YAML files (declarative approach)
- connect deployments together (networking + DNS)
- switch runtime from Docker to CRI-O

## Tiny glossary (for this chapter)
- **Container**: packaged app + dependencies
- **Orchestration**: managing many containers automatically
- **K8s/Kubernetes**: container orchestration platform
- **Runtime**: software that actually runs the containers