# Three-Tier Chat Application Deployment on Kubernetes

## Project Overview
This project demonstrates the deployment of a three-tier chat application on Kubernetes using a microservices-based architecture. The application consists of:

Frontend – User interface for interacting with the chat application
Backend – API layer handling authentication, messaging, and business logic
MongoDB – Database layer for storing users, messages, and application data

The application was containerized and deployed on a Kubernetes cluster with proper networking and persistent storage. Below image shows the flow of deployment followed for the chat application:

![Three-Tier-Chat-App-K8s-Deployment](Image/k8s-3-tier-deployment.jpg)

---

## Components Deployed

### 1. Frontend Tier
* Built Docker image for frontend application
* Pushed image to Docker Hub
* Created Kubernetes Deployment
* Exposed application internally using Service

### 2. Backend Tier
* Built Docker image for backend API
* Pushed image to Docker Hub
* Created Kubernetes Deployment
* Configured environment variables for MongoDB connectivity
* Exposed backend using Kubernetes Service

### 3. Database Tier (MongoDB)
* Used official MongoDB container image
* Created Deployment for MongoDB
* Created Service for internal communication
* Attached persistent storage using PV and PVC

---

## Deployment Steps with Screenshots

### Step 1

We will create and push Docker images from the already provided Dockerfiles to repositories created on Docker Hub. Below commands will be used:

```
docker build -t asadjvd/chatapp-backend:latest .
docker push asadjvd/chatapp-backend:latest
docker build -t asadjvd/chatapp-frontend:latest .
docker push asadjvd/chatapp-backend:latest
```

![Docker-Repository](Image/docker-repo.PNG)

---

### Step 2 

To carry out deployment of chat application on Kubernetes, I had made use of Minikube. Once Minikube is up and running we will start with the deployment. First step I had carried out was to create a namespace where we would deploy all our Kubernetes resources. The Kubernetes manifests are present in directory Kubernetes-Projects/full-stack_chatApp/Kubernetes/. Below is the command and screenshot:

```
kubectl apply -f namespace.yml
```

![Directory-Path](Image/directory-path.PNG)

![Chat-App-Namespace](Image/namespace.PNG)

---

### Step 3 

Created Persistent Volume (PV) and Persistent Volume Claim (PVC) and then mounted them to MongoDB container for durable storage. Without persistent storage MongoDB data would be lost if pod restarts. By having PV and PVC we would ensure database data survives incase a pod gets recreated. Below are the commands used to create PV and PVC ane the screenshots of the manifests used to create them:

```
kubectl apply -f mongodb-pv.yml
kubectl apply -f mongodb-pv.yml
```

![MongoDB-PV](Image/mongodb-pv.PNG)

![MongoDB-PVC](Image/mongodb-pvc.PNG)

![MongoDB-PV-PVC-Deployment](Image/deploy2.PNG)

---

### Step 4





