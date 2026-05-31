# OpenTelemetry Demo Project Deployment on Amazon EKS 

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

The application can be exposed directly using a Kubernetes Service of type LoadBalancer. For this purpose we will change the service type to **LoadBalancer** from **ClusterIP** in service named **opentelemetry-demo-frontendproxy**

Example:

```yaml
apiVersion: v1
kind: Service
spec:
  type: LoadBalancer
```

```bash
kubectl edit svc opentelemetry-demo-frontendproxy
```

Kubernetes provisions an AWS Load Balancer and assigns a public endpoint.

Retrieve the external address:

```bash
kubectl get svc | grep frontendproxy
```

Access the application using the generated DNS name.

## Access Method 2: AWS Load Balancer Controller + Ingress

A more production-oriented deployment uses the AWS Load Balancer Controller with Kubernetes Ingress resources.

### Install AWS Load Balancer Controller
 
#### Set Cluster Name:

```bash
export cluster_name=demo-cluster
```

This defines the EKS cluster name used in all following commands.

---

#### Configure IAM OIDC Provider:

```bash
oidc_id=$(aws eks describe-cluster \
  --name $cluster_name \
  --query "cluster.identity.oidc.issuer" \
  --output text | cut -d '/' -f 5)
```

##### What this does:

- Fetches the cluster's OIDC issuer URL
- Extracts the OIDC ID required for IAM integration

---

#### Check if OIDC Provider Already Exists

```bash
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

##### Purpose:

- Checks if IAM OIDC provider is already configured
- Avoids duplicate setup

---

#### If Not Exists → Create OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster $cluster_name \
  --approve
```

##### Why OIDC is needed:

- Enables EKS to use IAM Roles for Service Accounts (IRSA)
- Required for ALB Controller authentication with AWS

---

#### Download the IAM policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

##### Purpose:

- Downloads official AWS Load Balancer Controller IAM permissions

---

#### Create the IAM policy in AWS:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

##### Purpose:

- Creates IAM policy in AWS
- Grants permissions for managing ALB resources

---

#### Create the IAM Service Account (IRSA):

```bash
eksctl create iamserviceaccount \
  --cluster <cluster-name> \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

##### What this does:

- Creates Kubernetes Service Account
- Links it to IAM Role using IRSA
- Grants ALB controller AWS permissions securely

---

#### Key Concept (IRSA Flow)

```text id="irsa_flow"
ALB Controller Pod
        ↓
Service Account (K8s)
        ↓
IAM Role (AWS)
        ↓
Permissions (ALB / EC2 / ELB APIs)
```

---

### Deploy ALB Controller

#### Add Helm Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
```

##### Purpose:

- Adds AWS official Helm chart repository

---

#### Update Helm Repo

```bash
helm repo update eks
```

##### Purpose:

- Ensures latest charts are available

---

#### Install the AWS Load Balancer Controller using Helm.

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
---

#### Key Configuration Explained

| Parameter | Meaning |
|------------|--------|
| `clusterName` | EKS cluster name |
| `serviceAccount.create=false` | Uses pre-created IRSA service account |
| `serviceAccount.name` | Kubernetes service account for controller |
| `region` | AWS region |
| `vpcId` | VPC where ALB will be created |

---

##### What gets deployed:

- AWS Load Balancer Controller Pod
- Watches Ingress resources
- Automatically provisions ALB/NLB in AWS

---

#### Verify Deployment

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

---

#### Deploy Ingress

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
kubectl get pods,svc,ingress -A
```

## End-to-End Flow

```text id="alb_flow"
1. OIDC Provider Setup
        ↓
2. IAM Policy Creation
        ↓
3. IAM Role + Service Account (IRSA)
        ↓
4. Helm Install ALB Controller
        ↓
5. Controller Watches Ingress
        ↓
6. AWS ALB is Provisioned Automatically
```

---

## Final Result

#### After setup:

✅ EKS can provision ALBs automatically  
✅ Ingress resources work with AWS load balancers  
✅ Secure IAM access via IRSA  
✅ No manual AWS load balancer configuration needed  

---

To access the web application get IP address that is resolved from ALB DNS name using following command:

```bash
nslookup <ALB-DNS-Name>
```

Once IP is acquired next map it to **example.com** which is defined in the Ingress file we had applied. The mapping of IP address would be done in hosts file present in C:\Windows\System32\drivers\etc on Windows.  

---

## Cleanup

Remove application resources:

```bash
kubectl delete -f frontendproxy/ingress.yaml
```

```bash
kubectl delete -f complete-deploy.yaml
```

Verify removal:

```bash
kubectl get pods,svc,ingress -A
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
