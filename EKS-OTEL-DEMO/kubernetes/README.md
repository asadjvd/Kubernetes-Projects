# OpenTelemetry Demo Deployment on Amazon EKS

## Overview

This project deploys the OpenTelemetry Demo application on an Amazon EKS cluster using Kubernetes manifests.

The application consists of multiple microservices that simulate a modern cloud-native e-commerce platform. Each service is deployed independently and exposed internally through Kubernetes Services.

The deployment demonstrates:

* Kubernetes Deployments
* Kubernetes Services
* Ingress Management
* AWS Load Balancer Controller Integration
* Microservices Communication
* Cloud-Native Application Deployment
* Amazon EKS Operations

## Project Structure

```text
kubernetes/
├── accounting/
├── ad/
├── cart/
├── checkout/
├── currency/
├── email/
├── flagd/
├── frauddetection/
├── frontend/
├── frontendproxy/
├── imageprovider/
├── kafka/
├── loadgenerator/
├── payment/
├── productcatalog/
├── quote/
├── recommendation/
├── shipping/
├── valkey/
├── serviceaccount.yaml
├── complete-deploy.yaml
└── iam_policy.json
```

Each microservice contains its own deployment and service manifests.

## Microservices

The application includes the following services:

| Service        | Purpose                          |
| -------------- | -------------------------------- |
| frontend       | User-facing web application      |
| frontendproxy  | Entry point for incoming traffic |
| productcatalog | Product information service      |
| recommendation | Product recommendation engine    |
| cart           | Shopping cart service            |
| checkout       | Checkout workflow                |
| payment        | Payment processing               |
| shipping       | Shipping calculations            |
| currency       | Currency conversion              |
| quote          | Quote generation                 |
| email          | Email notifications              |
| ad             | Advertisement service            |
| accounting     | Accounting service               |
| frauddetection | Fraud detection service          |
| imageprovider  | Product image provider           |
| kafka          | Event streaming platform         |
| valkey         | Caching layer                    |
| flagd          | Feature flag management          |
| loadgenerator  | Simulated user traffic           |

## Prerequisites

Before deployment, ensure:

* Amazon EKS cluster is operational
* kubectl is configured
* Worker nodes are in Ready state
* AWS Load Balancer Controller is installed (for Ingress deployments)

Verify cluster connectivity:

```bash
kubectl get nodes
```

## Deployment

Deploy all application components using the consolidated manifest:

```bash
kubectl apply -f complete-deploy.yaml
```

Verify deployments:

```bash
kubectl get deployments
```

Verify pods:

```bash
kubectl get pods
```

Verify services:

```bash
kubectl get svc
```

## Access Method 1: LoadBalancer Service

The application can be exposed directly using a Kubernetes Service of type LoadBalancer.

Example:

```yaml
apiVersion: v1
kind: Service
spec:
  type: LoadBalancer
```

Kubernetes provisions an AWS Load Balancer and assigns a public endpoint.

Retrieve the external address:

```bash
kubectl get svc
```

Access the application using the generated DNS name.

## Access Method 2: AWS Load Balancer Controller + Ingress

A more production-oriented deployment uses the AWS Load Balancer Controller with Kubernetes Ingress resources.

### Install AWS Load Balancer Controller

Create the IAM policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

Create the IAM policy in AWS:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

Create the IAM Service Account (IRSA):

```bash
eksctl create iamserviceaccount \
  --cluster <cluster-name> \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn <policy-arn> \
  --approve
```

Install the controller using Helm.

### Deploy Ingress

Apply the ingress manifest:

```bash
kubectl apply -f frontendproxy/ingress.yaml
```

Verify ingress:

```bash
kubectl get ingress
```

Verify ALB creation:

```bash
kubectl describe ingress
```

AWS Load Balancer Controller automatically provisions an Application Load Balancer (ALB) and routes traffic to the frontend application.

## Validation

Verify all workloads:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

Check application availability:

```bash
kubectl get ingress
```

Open the generated ALB DNS name in a browser.

## Cleanup

Remove application resources:

```bash
kubectl delete -f complete-deploy.yaml
```

Verify removal:

```bash
kubectl get pods
```

## Skills Demonstrated

* Amazon EKS Administration
* Kubernetes Workload Management
* Kubernetes Services
* Kubernetes Ingress
* AWS Load Balancer Controller
* IAM Roles for Service Accounts (IRSA)
* Microservices Deployment
* Cloud-Native Architecture
* Application Exposure and Traffic Routing
* Infrastructure and Platform Operations
