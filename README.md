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

## Step 1️⃣. Create Docker image for the sample app

```bash
docker login
docker build -t <username>/sample-app:latest .
docker push <username>/sample-app:latest
```

> ⚠️ **Important:** By default, Minikube cannot detect Docker images built on your host machine when creating a Deployment! If the image is not available inside Minikube, Kubernetes will show an `ImagePullBackOff` error. To avoid this, either push your image to Docker Hub or build it inside Minikube’s Docker environment.

<br/>

Sample output:

<img width="1585" height="546" alt="image" src="https://github.com/user-attachments/assets/c6ed439d-2482-47e1-9f31-b8d340249ce3" />

<img width="1572" height="489" alt="image" src="https://github.com/user-attachments/assets/a95c790e-fdc3-4324-a557-f3fdd96e064a" />

<img width="1584" height="677" alt="image" src="https://github.com/user-attachments/assets/17dbc092-394f-4c93-8f3d-745a919f8de3" />


---

## Step 2️⃣. Start the Minikube cluster with the default driver

```bash
minikube start
```

Sample output:

<img width="1603" height="372" alt="image" src="https://github.com/user-attachments/assets/e591c2df-f446-4dbd-ba54-dac497232fd3" />

---

## Step 3️⃣. Create a Deployment

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
          image: fmfahathdev/python-demo-img:v1
          ports:
            - containerPort: 8000

```

Apply the Deployment configuration:

```bash
 kubectl apply -f deployment.yaml
```

> ⚠️ **Note:** In the `deployment.yaml` file, replace `image: fmfahathdev/python-demo-img:v1` with your own Docker Hub image: `<dockerhub-username>/image-name:tag`

Now that the cluster is created, it currently has:

- **2 running Pods:** `sample-python-app`
- **1 Deployment:** `sample-python-app`
- **1 ReplicaSet:** `sample-python-app`

Sample output:

<img width="1589" height="521" alt="image" src="https://github.com/user-attachments/assets/6c5cc35f-bdd2-41e9-8f56-4906aa3b4899" />


---

## Step 4️⃣. Self-Healing Test (ReplicaSet Behavior)

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

<img width="1576" height="433" alt="image" src="https://github.com/user-attachments/assets/5b442b7c-6441-48b6-bb0b-77e02c7538f6" />


---

## Step 5️⃣. Create a Service of type **NodePort**

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

<img width="1580" height="538" alt="image" src="https://github.com/user-attachments/assets/a66100cb-2fd6-468a-90f9-2f9a2a991b96" />

---

## Step 6️⃣: Accessing the Application Over the Network

### 🔹 Method 1: Accessing Using the Pod’s Cluster IP  *(Inside the Minikube cluster)*

**Steps:**

```bash
minikube ssh
curl -L http://10.244.0.31:8000/demo
```

**output**: The HTML output of the application will be displayed in the terminal.

Sample output:

<img width="1586" height="772" alt="image" src="https://github.com/user-attachments/assets/ff906a60-1eee-47e0-b825-3f7083f830e0" />

### 🔹 Method 2: Accessing Using the Service’s Cluster IP  *(Inside the Minikube cluster)*

**Steps:**

```bash
minikube ssh
curl -L http://10.111.156.103:80/demo
```

**output**: The HTML output of the application will be displayed in the terminal.

Sample output:

<img width="1577" height="754" alt="image" src="https://github.com/user-attachments/assets/074cf066-533a-4a89-9bff-e82c6a7c116a" />

### 🔹 Method 3: Accessing Using Minikube NodePort IP *(Outside the cluster but inside the local / organization network)*

**Steps:**

```bash
curl -L http://192.168.49.2:30007/demo
```

**output**: The HTML output of the application will be displayed in the terminal.

Sample output:

<img width="1570" height="769" alt="image" src="https://github.com/user-attachments/assets/a158daa6-304c-4808-b2d1-b105e4bedab4" />

### 🔹 Method 4: Accessing via Web Browser Using Minikube NodePort

**Steps:**

```bash
http://192.168.49.2:30007/demo
```

**output**: The application homepage will be displayed in the browser.

Sample output:

<img width="1024" height="768" alt="Screenshot from 2026-02-03 02-41-14" src="https://github.com/user-attachments/assets/065e688d-86c2-4221-87f4-da8556568166" />





