# Three-Tier Chat Application Deployment on Kubernetes

## Project Overview
This project demonstrates the deployment of a three-tier chat application on Kubernetes using a microservices-based architecture. The application consists of:

Frontend – User interface for interacting with the chat application
Backend – API layer handling authentication, messaging, and business logic
MongoDB – Database layer for storing users, messages, and application data

The application was containerized and deployed on a Kubernetes cluster with proper networking and persistent storage. Below image shows the flow of deployment followed for the chat application:

![Three-Tier-Chat-App-K8s-Deployment](Image/k8s-3-tier-deployment.jpg)

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



