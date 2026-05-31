# Architecture

## Solution Overview

This project provisions an Amazon EKS cluster for the OpenTelemetry Project using Terraform within a custom VPC. The infrastructure is distributed across multiple Availability Zones to improve availability and fault tolerance.

The architecture follows AWS best practices by deploying Kubernetes worker nodes in private subnets while exposing public-facing resources through public subnets.

## High-Level Architecture

```text
            Internet
                │
                ▼ 
┌──────────────────────────────┐
│      Internet Gateway        │
└──────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────┐
│                    VPC                      │
│                10.0.0.0/16                  │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │         Availability Zone A         │    │
│  │                                     │    │
│  │ Public Subnet                       │    │
│  │ 10.0.3.0/24                         │    │
│  │ ┌─────────────┐                     │    │
│  │ │ NAT Gateway │                     │    │
│  │ └─────────────┘                     │    │
│  │                                     │    │
│  │ Private Subnet                      │    │
│  │ 10.0.1.0/24                         │    │
│  │ ┌─────────────────────────┐         │    │
│  │ │ EKS Worker Nodes        │         │    │
│  │ └─────────────────────────┘         │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │         Availability Zone B         │    │
│  │                                     │    │
│  │ Public Subnet                       │    │
│  │ 10.0.4.0/24                         │    │
│  │ ┌─────────────┐                     │    │
│  │ │ NAT Gateway │                     │    │
│  │ └─────────────┘                     │    │
│  │                                     │    │
│  │ Private Subnet                      │    │
│  │ 10.0.2.0/24                         │    │
│  │ ┌─────────────────────────┐         │    │
│  │ │ EKS Worker Nodes        │         │    │
│  │ └─────────────────────────┘         │    │
│  └─────────────────────────────────────┘    │
│                                             │
│         Amazon EKS Control Plane            │
└─────────────────────────────────────────────┘
```

## Components

### VPC

A dedicated Virtual Private Cloud provides network isolation for all resources.

| Component        | CIDR        |
| ---------------- | ----------- |
| VPC              | 10.0.0.0/16 |
| Private Subnet A | 10.0.1.0/24 |
| Private Subnet B | 10.0.2.0/24 |
| Public Subnet A  | 10.0.3.0/24 |
| Public Subnet B  | 10.0.4.0/24 |

### Public Subnets

Public subnets host internet-facing networking resources.

Resources:

* Internet Gateway connectivity
* NAT Gateways
* Future Load Balancers

### Private Subnets

Private subnets host the EKS worker nodes.

Benefits:

* No direct internet exposure
* Reduced attack surface
* Secure workload deployment

### NAT Gateways

A NAT Gateway is deployed in each public subnet to allow resources within private subnets to access the internet for:

* Pulling container images
* Downloading updates
* Accessing AWS services

while remaining inaccessible from the public internet.

### Amazon EKS

The EKS control plane manages:

* Kubernetes API Server
* Scheduler
* Controller Manager
* etcd

The control plane is fully managed by AWS.

### Managed Node Group

Worker nodes are deployed using Amazon EKS Managed Node Groups.

Configuration:

| Setting            | Value     |
| ------------------ | --------- |
| Instance Type      | t3.medium |
| Capacity Type      | On-Demand |
| Desired Nodes      | 2         |
| Minimum Nodes      | 1         |
| Maximum Nodes      | 4         |
| Kubernetes Version | 1.30      |

## Traffic Flow

### Outbound Traffic

```text
Pod
 ↓
Worker Node
 ↓
Private Subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

### Inbound Traffic (Future Application Deployments)

```text
Internet
 ↓
Application Load Balancer
 ↓
Kubernetes Service
 ↓
Pod
```

## Terraform State Management

Infrastructure state is managed remotely using:

* Amazon S3 for Terraform state storage
* Amazon DynamoDB for state locking

Benefits:

* Centralized state management
* Team collaboration support
* Protection against concurrent modifications
* Improved reliability and recoverability

## Design Considerations

* Multi-AZ deployment for high availability
* Private worker nodes for enhanced security
* Managed Kubernetes control plane
* Scalable node group configuration
* Infrastructure as Code using Terraform
* Remote state management using S3 and DynamoDB
* Modular Terraform architecture for maintainability
