# Chapter 5 — Build a Custom App, Push to Docker Hub, Deploy, YAML, Rollouts, Dashboard, and Runtime Switch

In this chapter we follow the full practical workflow exactly as taught: build a small app, dockerize it, push it, deploy it, expose it, scale it, update it, move to YAML, connect two deployments, and finally switch runtime to CRI‑O.

## Part A — Build a simple Node.js + Express app

Goal: return a message that includes the hostname, so we can see which pod answered the request.

### 1) Create a project folder
On your machine:

```bash
mkdir k8s
cd k8s
```

Open it in VS Code:
```bash
code .
```

If `code` is not found, in VS Code open Command Palette and install “Shell command: Install 'code' command in PATH”.

### 2) Create the app folder
```bash
mkdir k8s-web-hello
cd k8s-web-hello
```

### 3) Initialize Node project + install Express
You need Node.js + npm installed on your computer.

```bash
npm init -y
npm install express
```

### 4) Create the server file: index.mjs
We use `.mjs` so we can use `import`.

Create `index.mjs`:

```js
import express from "express";
import os from "os";

const app = express();
const port = 3000;

app.get("/", (req, res) => {
  const message = `Hello from ${os.hostname()}`;
  console.log(message);
  res.send(message);
});

app.listen(port, () => {
  console.log(`Web server is listening at port ${port}`);
});
```

### 5) Update package.json to add a start script
Open `package.json` and set:

```json
"scripts": {
  "start": "node index.mjs"
}
```

Now `npm start` would run the server.

Note: you will not run it locally for this course; we will run it inside containers.

## Part B — Dockerize the app (Dockerfile)

Install Docker Desktop and start Docker.

Create `Dockerfile`:

```dockerfile
FROM node:alpine

WORKDIR /app

EXPOSE 3000

COPY package.json package-lock.json ./
RUN npm install

COPY . .

CMD ["npm", "start"]
```

This does:
- uses a small Node base image (alpine)
- copies package files and installs dependencies
- copies the source file
- starts the server

## Part C — Build image and push to Docker Hub

Create a Docker Hub account if you don’t have one.

Build the image (replace `<your_dockerhub_user>`):
```bash
docker build . -t <your_dockerhub_user>/k8s-web-hello
```

Check the image exists:
```bash
docker images | grep k8s-web-hello
```

Login to Docker Hub:
```bash
docker login
```

Push:
```bash
docker push <your_dockerhub_user>/k8s-web-hello
```

Now the image is public under your account.

## Part D — Deploy the custom image in Kubernetes + expose it

Make sure Minikube is running:
```bash
minikube status
```

Create deployment:
```bash
kubectl create deployment k8s-web-hello --image=<your_dockerhub_user>/k8s-web-hello
kubectl get pods
```

Wait until the pod is Running (image pull takes time).

Expose it as a service (ClusterIP first):
```bash
kubectl expose deployment k8s-web-hello --port=3000
kubectl get svc
```

ClusterIP is internal, so test from inside the node:

```bash
ssh docker@<MINIKUBE_IP>
# password: tcuser
curl http://<CLUSTER_IP>:3000
```

You should see:
`Hello from <pod-hostname>`

### Scale the deployment
Scale to 4:
```bash
kubectl scale deployment k8s-web-hello --replicas=4
kubectl get pods -o wide
```

Test load-balancing by curling multiple times from inside the node and watching hostname change.

## Part E — Expose to your laptop (NodePort)

Delete old service:
```bash
kubectl delete svc k8s-web-hello
```

Create NodePort service:
```bash
kubectl expose deployment k8s-web-hello --type=NodePort --port=3000
kubectl get svc
```

Open it in browser easily:
```bash
minikube service k8s-web-hello
```

To print only URL:
```bash
minikube service k8s-web-hello --url
```

Refreshing the browser should show different hostnames (requests going to different pods).

## Part F — LoadBalancer (cloud concept; Minikube shows EXTERNAL-IP Pending)

Delete NodePort service:
```bash
kubectl delete svc k8s-web-hello
```

Create LoadBalancer service:
```bash
kubectl expose deployment k8s-web-hello --type=LoadBalancer --port=3000
kubectl get svc
```

In Minikube, EXTERNAL-IP may show Pending, but `minikube service k8s-web-hello` still gives a working URL.

## Part G — Rolling updates (new image version)

Now we update the application so we can see a new version.

Edit `index.mjs` and change the response to include a version:

```js
const message = `Version 2: Hello from ${os.hostname()}`;
```

Build a new tagged image:
```bash
docker build . -t <your_dockerhub_user>/k8s-web-hello:2.0.0
docker push <your_dockerhub_user>/k8s-web-hello:2.0.0
```

Update the deployment image (rolling update):
```bash
kubectl set image deployment/k8s-web-hello k8s-web-hello=<your_dockerhub_user>/k8s-web-hello:2.0.0
kubectl rollout status deployment/k8s-web-hello
```

List pods and notice ages reset:
```bash
kubectl get pods
```

Open the service again and confirm “Version 2” appears.

### Roll back by switching image back to latest
```bash
kubectl set image deployment/k8s-web-hello k8s-web-hello=<your_dockerhub_user>/k8s-web-hello:latest
kubectl rollout status deployment/k8s-web-hello
```

## Part H — Self-healing demo (delete a pod)

If replicas=4 and you delete one pod, Kubernetes recreates it:

```bash
kubectl get pods
kubectl delete pod <one_pod_name>
kubectl get pods
```

You will see a new pod created automatically.

## Part I — Kubernetes Dashboard (Minikube)

Start dashboard:
```bash
minikube dashboard
```

In the UI you can inspect:
- deployments
- replica sets (you may see old RS with 0 pods after rollouts)
- pods (events, IPs, node)
- services
- nodes

Stop dashboard proxy with Ctrl+C.

## Part J — Declarative approach (YAML) instead of imperative commands

Imperative is fast for demos. In real work, you store YAML in Git and apply it.

### 1) Create deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-web-hello
spec:
  replicas: 5
  selector:
    matchLabels:
      app: k8s-web-hello
  template:
    metadata:
      labels:
        app: k8s-web-hello
    spec:
      containers:
        - name: k8s-web-hello
          image: <your_dockerhub_user>/k8s-web-hello:latest
          ports:
            - containerPort: 3000
          resources:
            limits:
              memory: "256Mi"
              cpu: "500m"
```

Apply it:
```bash
kubectl apply -f deployment.yaml
kubectl get deploy
kubectl get pods
```

Change replicas by editing `replicas:` and applying again.

### 2) Create service.yaml (LoadBalancer example)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: k8s-web-hello
spec:
  type: LoadBalancer
  selector:
    app: k8s-web-hello
  ports:
    - port: 3030
      targetPort: 3000
```

Apply:
```bash
kubectl apply -f service.yaml
kubectl get svc
minikube service k8s-web-hello
```

### 3) Delete by files
```bash
kubectl delete -f deployment.yaml -f service.yaml
```

### 4) Delete everything quickly (shortcut used in the course)
```bash
kubectl delete all --all
```

## Part K — Connect two deployments together (web calls nginx by service name)

Now we create two deployments:
1) A custom web app that has an endpoint `/nginx` which calls `http://nginx`
2) A plain nginx deployment exposed internally with a ClusterIP service named `nginx`

### 1) Create a second app folder (web that calls nginx)

Copy your project folder and rename:
- `k8s-web-hello` → `k8s-web2-nginx`

In that folder:
- install one more dependency:
```bash
npm install node-fetch
```

Create `index.mjs` for the second app:

```js
import express from "express";
import os from "os";
import fetch from "node-fetch";

const app = express();
const port = 3000;

app.get("/", (req, res) => {
  const message = `Hello from ${os.hostname()}`;
  console.log(message);
  res.send(message);
});

app.get("/nginx", async (req, res) => {
  const response = await fetch("http://nginx");
  const body = await response.text();
  res.send(body);
});

app.listen(port, () => {
  console.log(`Web server is listening at port ${port}`);
});
```

Build + push image:
```bash
docker build . -t <your_dockerhub_user>/k8s-web2-nginx
docker push <your_dockerhub_user>/k8s-web2-nginx
```

### 2) Create a combined YAML for web2 service + deployment (one file)

Create `k8s-web2-nginx.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: k8s-web2-nginx
spec:
  type: LoadBalancer
  selector:
    app: k8s-web2-nginx
  ports:
    - port: 3030
      targetPort: 3000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-web2-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: k8s-web2-nginx
  template:
    metadata:
      labels:
        app: k8s-web2-nginx
    spec:
      containers:
        - name: k8s-web2-nginx
          image: <your_dockerhub_user>/k8s-web2-nginx:latest
          ports:
            - containerPort: 3000
          resources:
            limits:
              memory: "256Mi"
              cpu: "500m"
```

### 3) Create nginx.yaml (ClusterIP service name MUST be nginx)

Create `nginx.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            limits:
              memory: "256Mi"
              cpu: "500m"
```

Apply both:
```bash
kubectl apply -f k8s-web2-nginx.yaml -f nginx.yaml
kubectl get deploy
kubectl get svc
kubectl get pods
```

Test:
```bash
minikube service k8s-web2-nginx
```

In browser:
- `/` returns “Hello from …”
- `/nginx` returns the nginx welcome HTML (proxied through your web app)

### 4) Prove DNS resolution inside the cluster

Pick a web2 pod and run:

```bash
kubectl exec -it <WEB2_POD_NAME> -- nslookup nginx
```

It should resolve to the ClusterIP of the nginx service.

Also test HTTP from inside the pod:
```bash
kubectl exec -it <WEB2_POD_NAME> -- wget -qO- http://nginx
```

Clean up:
```bash
kubectl delete -f nginx.yaml -f k8s-web2-nginx.yaml
```

## Part L — Switch container runtime from Docker to CRI‑O

Now we prove Kubernetes can run without Docker runtime inside the node.

Stop and delete current cluster:
```bash
minikube stop
minikube delete
```

Start a new cluster with CRI‑O runtime (use your driver):
```bash
minikube start --driver=virtualbox --container-runtime=cri-o
```

Check status:
```bash
minikube status
```

SSH into node:
```bash
minikube ip
ssh docker@<MINIKUBE_IP>
# password: tcuser
```

Now Docker commands should fail:
```bash
docker ps
```

Instead, list containers using `crictl`:
```bash
sudo crictl ps
```

Deploy your YAML files again (web2 + nginx) and test `/nginx` endpoint as before. It should work the same, but now containers are managed by CRI‑O.

That’s the full course workflow.
