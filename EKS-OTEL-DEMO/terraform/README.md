# Terraform AWS EKS Cluster for OTEL Demo Project

## Overview

This project provisions a production-style Amazon EKS (Elastic Kubernetes Service) cluster on AWS using Terraform.

The infrastructure is deployed into a custom VPC spanning multiple Availability Zones and includes both public and private subnets. Worker nodes are deployed in private subnets for improved security while maintaining outbound internet access through NAT Gateways.

Terraform remote state management is configured using an S3 backend with DynamoDB state locking to enable safe collaboration and prevent concurrent state modifications.

## Features

* Custom VPC with DNS support enabled
* Multi-AZ deployment across two Availability Zones
* Public and private subnet architecture
* Internet Gateway for public connectivity
* NAT Gateways for outbound access from private subnets
* Amazon EKS Cluster (Kubernetes v1.30)
* Managed Node Groups
* Terraform Remote Backend (S3 + DynamoDB Locking)
* Modular Terraform code structure
* Infrastructure as Code (IaC) approach

## Infrastructure Components

### Networking

* VPC CIDR: `10.0.0.0/16`
* Public Subnets:

  * `10.0.3.0/24`
  * `10.0.4.0/24`
* Private Subnets:

  * `10.0.1.0/24`
  * `10.0.2.0/24`

### EKS Cluster

* Cluster Name: `tf-eks-cluster`
* Kubernetes Version: `1.30`
* Managed Node Group:

  * Instance Type: `t3.medium`
  * Capacity Type: `ON_DEMAND`
  * Desired Capacity: `2`
  * Minimum Capacity: `1`
  * Maximum Capacity: `4`

## Prerequisites

Before deploying the infrastructure, ensure the following tools are installed:

* Terraform
* AWS CLI
* kubectl
* AWS Account with appropriate permissions

Configure AWS credentials:

```bash
aws configure
```

## Deployment

### Clone Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### Initialize Terraform

```bash
terraform init
```

### Review Planned Changes

```bash
terraform plan
```

### Deploy Infrastructure

```bash
terraform apply
```

Approve the deployment when prompted.

### Configure kubectl

After the EKS cluster is created, update the kubeconfig file:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name tf-eks-cluster
```

### Verify Cluster Access

```bash
kubectl get nodes
```

Expected output should display the managed worker nodes in a `Ready` state.

## Useful Commands

### View Cluster Information

```bash
kubectl cluster-info
```

### View Nodes

```bash
kubectl get nodes -o wide
```

### View System Pods

```bash
kubectl get pods -A
```

## Cleanup

To destroy all infrastructure and save cost:

```bash
terraform destroy --auto-approve
```

## Notes

* Terraform state is stored remotely in Amazon S3.
* State locking is handled through DynamoDB.
* Worker nodes are deployed in private subnets.
* NAT Gateways provide outbound internet connectivity for private resources.
* The cluster is designed as a foundation for deploying containerized applications and Kubernetes workloads.
