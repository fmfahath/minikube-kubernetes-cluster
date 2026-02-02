# Create a Kubernetes Cluster using Minikube

This project demonstrates how to create and manage a local Kubernetes cluster using **Minikube**.  
It is ideal for learning Kubernetes basics, testing deployments, and practicing container orchestration in a local environment.

<img width="1536" height="1024" alt="ChatGPT Image Feb 2, 2026, 08_14_14 PM" src="https://github.com/user-attachments/assets/bfc09050-7742-4f63-8e3e-81ad8aa24167" />


---

## 📌 Project Overview

Minikube allows you to run Kubernetes locally on your machine.  
In this project, we:
- Create a Docker image for the application
- Set up Minikube
- Create a Kubernetes cluster
- Deploy a sample application to the cluster
- Create a Service of type **NodePort**
- Verify Pods, Services, and overall cluster status

---

## 🛠️ Prerequisites

Ensure the following tools are installed on your system:

- Docker
- Minikube
- kubectl
- Virtualization enabled (if not using Docker driver)

---

## 📂 Project Structure

```bash
.
├── DevOps
├── Dockerfile
├── deployment.yaml
├── service.yaml
└── README.md
```
---

## ⚙️ Installation & Setup

### 1. Create Docker image for the sample app

```bash
docker login
docker build -t <username>/sample-app:latest .
docker push <username>/sample-app:latest
```

> ⚠️ **Important:** By default, Minikube cannot detect Docker images built on your host machine when creating a Deployment! If the image is not available inside Minikube, Kubernetes will show an `ImagePullBackOff` error. To avoid this, either push your image to Docker Hub or build it inside Minikube’s Docker environment.

<br/>

Sample output:

<img width="1573" height="752" alt="image" src="https://github.com/user-attachments/assets/b05b5d2f-0f29-4ec1-81b1-7c2cd01afa24" />

### 2. Start the Minikube cluster with the default driver

```bash
minikube start
```

Sample output:

<img width="1593" height="366" alt="image" src="https://github.com/user-attachments/assets/876f88b3-1e2f-4206-b48f-6557f74e3f09" />

### 3️⃣ Create a Deployment

Deploy your application to the Kubernetes cluster using the `deployment.yaml` file.

`deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-python-app
  labels:
    app: sample-python-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-python-app
  template:
    metadata:
      labels:
        app: sample-python-app
    spec:
      containers:
        - name: python-app
          image: fmfahathdev/python-demo:v1
          ports:
            - containerPort: 8000

```

Apply the Deployment configuration:

```bash
 kubectl apply -f deployment.yaml
```

> ⚠️ **Note:** In the `deployment.yaml` file, replace `image: fmfahathdev/python-demo:v1` with your own Docker Hub image: `<dockerhub-username>/image-name:tag`

Now that the cluster is created, it currently has:

- **2 running Pods:** `sample-python-app`
- **1 Deployment:** `sample-python-app`
- **1 ReplicaSet:** `sample-python-app`

Sample output:

<img width="1586" height="458" alt="image" src="https://github.com/user-attachments/assets/1f58b2ca-1972-4773-934a-08abfa7de7bd" />

---

### 4. Self-Healing Test (ReplicaSet Behavior)

To verify Kubernetes self-healing, delete one of the running Pods.  
The ReplicaSet will automatically create a new Pod to maintain the desired number of replicas.

**Delete a Pod:**

```bash
kubectl delete pod sample-python-app-5df9589948-lwvgt
```
To verify this, use the watch command 

```bash
 kubectl get pods -w
```

Sample output:

<img width="1558" height="440" alt="image" src="https://github.com/user-attachments/assets/8d63e696-fd5b-4524-b3af-202c3b958762" />

---

### 5️⃣ Create a Service of type **NodePort**

Create a Kubernetes Service to expose the application outside the cluster using a **NodePort** service.

`service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-python-app-service
spec:
  type: NodePort
  selector:
    app: sample-python-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007

```

Apply the Service configuration:

```bash
kubectl apply -f service.yaml
```

Now, new service `sample-python-app-service` created in the cluster

Sample output:

<img width="1592" height="434" alt="image" src="https://github.com/user-attachments/assets/c0753a19-a469-4302-8142-022308735768" />



