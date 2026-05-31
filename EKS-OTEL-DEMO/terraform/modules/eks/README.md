# EKS Module - Code Walkthrough

This module provisions an Amazon EKS cluster along with managed node groups and the required IAM roles and policies.

---

## Architecture Overview

```text
┌─────────────────────┐
│   IAM Cluster Role  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│     EKS Cluster     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Managed Node Groups │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   EC2 Worker Nodes  │
└─────────────────────┘
```

---

# 1. Create IAM Role for EKS Cluster

```hcl
resource "aws_iam_role" "cluster" {
  name = "${var.cluster_name}-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
    }]
  })
}
```

### Purpose

Creates an IAM role that Amazon EKS uses to manage the Kubernetes control plane.

### Key Components

| Attribute | Description |
|------------|-------------|
| `name` | Name of the IAM role |
| `assume_role_policy` | Trust relationship allowing EKS service to assume the role |

### Why It Is Needed

The EKS control plane requires AWS permissions to:

- Manage cluster resources
- Communicate with AWS services
- Create and manage networking components

---

# 2. Attach IAM Policy to Cluster Role

```hcl
resource "aws_iam_role_policy_attachment" "cluster_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.cluster.name
}
```

### Purpose

Attaches the AWS-managed EKS Cluster Policy to the cluster IAM role.

### Permissions Granted

- Cluster lifecycle management
- Kubernetes control plane operations
- AWS resource interaction required by EKS

### Why It Is Needed

Without this policy, the EKS service cannot properly create or manage the cluster.

---

# 3. Create the EKS Cluster

```hcl
resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  version  = var.cluster_version
  role_arn = aws_iam_role.cluster.arn

  vpc_config {
    subnet_ids = var.subnet_ids
  }

  depends_on = [
    aws_iam_role_policy_attachment.cluster_policy
  ]
}
```

### Purpose

Creates the Kubernetes control plane in AWS.

### Key Components

| Attribute | Description |
|------------|-------------|
| `name` | EKS cluster name |
| `version` | Kubernetes version |
| `role_arn` | Cluster IAM role |
| `subnet_ids` | Subnets used by the cluster |

### Dependency

```hcl
depends_on = [
  aws_iam_role_policy_attachment.cluster_policy
]
```

Ensures the IAM policy is attached before cluster creation begins.

### Outcome

After deployment:

- Kubernetes API Server is created
- Control plane becomes available
- Worker nodes can later join the cluster

---

# 4. Create IAM Role for Worker Nodes

```hcl
resource "aws_iam_role" "node" {
  name = "${var.cluster_name}-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}
```

### Purpose

Creates an IAM role for EC2 instances that will act as Kubernetes worker nodes.

### Why It Is Needed

Worker nodes require AWS permissions to:

- Join the EKS cluster
- Communicate with the control plane
- Pull container images
- Configure networking

---

# 5. Attach Required Policies to Worker Nodes

```hcl
resource "aws_iam_role_policy_attachment" "node_policy" {
  for_each = toset([
    "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
    "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  ])

  policy_arn = each.value
  role       = aws_iam_role.node.name
}
```

### Purpose

Attaches required AWS-managed policies to the worker node role.

### Policies Explained

#### AmazonEKSWorkerNodePolicy

Allows nodes to:

- Register with the cluster
- Communicate with Kubernetes control plane
- Access EKS APIs

#### AmazonEKS_CNI_Policy

Allows the AWS VPC CNI plugin to:

- Allocate IP addresses
- Manage ENIs
- Configure pod networking

#### AmazonEC2ContainerRegistryReadOnly

Allows nodes to:

- Pull Docker images from Amazon ECR
- Authenticate with ECR repositories

---

# 6. Create Managed Node Groups

```hcl
resource "aws_eks_node_group" "main" {
  for_each = var.node_groups

  cluster_name    = aws_eks_cluster.main.name
  node_group_name = each.key
  node_role_arn   = aws_iam_role.node.arn
  subnet_ids      = var.subnet_ids

  instance_types = each.value.instance_types
  capacity_type  = each.value.capacity_type

  scaling_config {
    desired_size = each.value.scaling_config.desired_size
    max_size     = each.value.scaling_config.max_size
    min_size     = each.value.scaling_config.min_size
  }

  depends_on = [
    aws_iam_role_policy_attachment.node_policy
  ]
}
```

### Purpose

Creates one or more managed EKS node groups.

### Dynamic Node Group Creation

```hcl
for_each = var.node_groups
```

Allows multiple node groups to be created using a single Terraform resource.

Example:

```hcl
node_groups = {
  general = {
    instance_types = ["t3.medium"]
    capacity_type  = "ON_DEMAND"

    scaling_config = {
      desired_size = 2
      min_size     = 1
      max_size     = 3
    }
  }

  spot = {
    instance_types = ["t3.large"]
    capacity_type  = "SPOT"

    scaling_config = {
      desired_size = 1
      min_size     = 1
      max_size     = 5
    }
  }
}
```

### Scaling Configuration

| Parameter | Purpose |
|------------|----------|
| `desired_size` | Number of nodes to maintain |
| `min_size` | Minimum node count |
| `max_size` | Maximum node count |

### Capacity Types

| Type | Description |
|--------|-------------|
| `ON_DEMAND` | Standard EC2 instances |
| `SPOT` | Discounted spare AWS capacity |

### Dependency

```hcl
depends_on = [
  aws_iam_role_policy_attachment.node_policy
]
```

Ensures worker node permissions are configured before node creation.

---

# Deployment Flow

```text
1. Create Cluster IAM Role
        │
        ▼
2. Attach AmazonEKSClusterPolicy
        │
        ▼
3. Create EKS Cluster
        │
        ▼
4. Create Node IAM Role
        │
        ▼
5. Attach Worker Node Policies
        │
        ▼
6. Create Managed Node Groups
        │
        ▼
7. Worker Nodes Join Cluster
```

---

# Result

After successful deployment:

✅ EKS Control Plane is provisioned

✅ IAM Roles and Policies are configured

✅ Managed Node Groups are created

✅ Worker Nodes join the cluster automatically

✅ Cluster is ready for Kubernetes workloads
