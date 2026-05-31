# VPC Module - Code Walkthrough

This module builds a fully functional AWS VPC networking layer for an EKS cluster, including public/private subnets, routing, and NAT gateways.

---

## Architecture Overview

```text
                ┌──────────────────────┐
                │   Internet Gateway   │
                └─────────┬────────────┘
                          │
                ┌─────────▼────────────┐
                │     Public Subnets   │
                │  (ALB / NAT Gateways)│
                └─────────┬────────────┘
                          │
                    NAT Gateway
                          │
                ┌─────────▼────────────┐
                │    Private Subnets   │
                │ (EKS Worker Nodes)    │
                └──────────────────────┘
```

---

# 1. Create VPC

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name                                        = "${var.cluster_name}-vpc"
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
  }
}
```

### Purpose

Creates the base Virtual Private Cloud (VPC) for all infrastructure.

### Key Features

| Feature | Purpose |
|----------|--------|
| `cidr_block` | Defines IP range for the VPC |
| `enable_dns_support` | Enables internal DNS resolution |
| `enable_dns_hostnames` | Allows EC2 hostname assignment |

### Kubernetes Tagging

```text
kubernetes.io/cluster/<cluster-name> = shared
```

This tag allows Kubernetes (EKS) to recognize and use the VPC.

---

# 2. Create Private Subnets

```hcl
resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name                                        = "${var.cluster_name}-private-${count.index + 1}"
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"           = "1"
  }
}
```

### Purpose

Creates multiple private subnets for backend workloads like EKS worker nodes.

### Key Features

- Uses `count` to create multiple subnets dynamically
- Distributed across availability zones
- No direct internet access

### Kubernetes Role Tag

```text
kubernetes.io/role/internal-elb = 1
```

Enables internal Kubernetes load balancers.

---

# 3. Create Public Subnets

```hcl
resource "aws_subnet" "public" {
  count             = length(var.public_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  map_public_ip_on_launch = true

  tags = {
    Name                                        = "${var.cluster_name}-public-${count.index + 1}"
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                    = "1"
  }
}
```

### Purpose

Creates public subnets used for internet-facing resources.

### Key Features

- Auto-assigns public IPs
- Used for Load Balancers and NAT Gateways
- Spread across AZs

### Kubernetes Role Tag

```text
kubernetes.io/role/elb = 1
```

Enables external load balancers.

---

# 4. Create Internet Gateway

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.cluster_name}-igw"
  }
}
```

### Purpose

Provides internet access to resources inside public subnets.

### Role

- Allows outbound/inbound internet traffic
- Required for NAT Gateway functionality

---

# 5. Create NAT Gateways and Elastic IPs

## Elastic IPs

```hcl
resource "aws_eip" "nat" {
  count  = length(var.public_subnet_cidrs)
  domain = "vpc"

  tags = {
    Name = "${var.cluster_name}-nat-${count.index + 1}"
  }
}
```

### Purpose

Allocates static public IPs for NAT Gateways.

---

## NAT Gateways

```hcl
resource "aws_nat_gateway" "main" {
  count         = length(var.public_subnet_cidrs)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "${var.cluster_name}-nat-${count.index + 1}"
  }
}
```

### Purpose

Allows private subnet instances to access the internet securely.

### Key Behavior

- Outbound internet access only
- No inbound access from internet
- Placed in public subnets

---

# 6. Create Route Tables

## Public Route Table

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "${var.cluster_name}-public"
  }
}
```

### Purpose

Routes all internet traffic directly via Internet Gateway.

---

## Private Route Tables

```hcl
resource "aws_route_table" "private" {
  count  = length(var.private_subnet_cidrs)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = {
    Name = "${var.cluster_name}-private-${count.index + 1}"
  }
}
```

### Purpose

Routes private subnet traffic through NAT Gateways.

### Key Benefit

- Keeps instances private
- Still allows outbound internet access

---

# 7. Associate Route Tables with Subnets

## Private Subnets Association

```hcl
resource "aws_route_table_association" "private" {
  count          = length(var.private_subnet_cidrs)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

### Purpose

Links each private subnet to its respective route table.

---

## Public Subnets Association

```hcl
resource "aws_route_table_association" "public" {
  count          = length(var.public_subnet_cidrs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

### Purpose

Links all public subnets to the shared public route table.

---

# Final Result

After deployment, the VPC provides:

✅ Highly available multi-AZ networking  
✅ Public subnets for load balancers and NAT  
✅ Private subnets for secure workloads (EKS nodes)  
✅ Internet Gateway for public access  
✅ NAT Gateways for secure outbound internet  
✅ Kubernetes-ready networking setup  

---

# Flow Summary

```text
1. VPC Created
      │
      ▼
2. Public & Private Subnets Created
      │
      ▼
3. Internet Gateway Attached
      │
      ▼
4. NAT Gateways Provisioned
      │
      ▼
5. Route Tables Configured
      │
      ▼
6. Subnets Associated with Routes
      │
      ▼
7. Ready for EKS Deployment
```
