# AWS EKS VPC CNI Plugin - Complete Guide

# Table of Contents

1. Introduction
2. What is AWS VPC CNI Plugin
3. EKS Networking Architecture
4. What is ENI
5. Primary IP vs Secondary IP
6. How Pods Get IP Addresses
7. Pod Communication Flow
8. Node Communication
9. Internet Communication
10. Cross Node Pod Communication
11. IP Assignment Flow
12. ENI Limits
13. Maximum Pods Calculation
14. IP Exhaustion
15. Prefix Delegation
16. Security Groups
17. Route Tables
18. NAT Gateway Flow
19. AWS VPC CNI Components
20. Useful Commands
21. Architecture Diagram
22. Summary

---

# 1. Introduction

Amazon EKS uses the AWS VPC CNI Plugin for Kubernetes networking.

Unlike traditional Kubernetes networking:

- Pods receive real VPC IP addresses
- Pods become directly reachable inside AWS VPC
- No overlay networking required

This provides:

- Better networking performance
- Low latency
- Native AWS routing
- Easy communication with AWS services

---

# 2. What is AWS VPC CNI Plugin

## Full Form

VPC CNI =
Virtual Private Cloud Container Network Interface

---

## Purpose

AWS VPC CNI Plugin is responsible for:

- Assigning IP addresses to Pods
- Managing ENIs
- Managing secondary IPs
- Connecting Pods to AWS VPC network

---

## Installed As

The plugin runs as:

```text
aws-node
```

DaemonSet inside:

```text
kube-system namespace
```

---

## Verify Plugin

```bash
kubectl get daemonset -n kube-system
```

### Output

```text
aws-node
```

---

# 3. EKS Networking Architecture

## Traditional Kubernetes Networking

Traditional Kubernetes uses overlay networks:

- Flannel
- Calico
- Weave

Pods communicate through virtual overlay tunnels.

---

## AWS EKS Networking

In EKS:

- Pods directly use VPC IPs
- AWS route tables handle traffic
- Native AWS networking is used
- No overlay tunnels required

---

## Benefits

- High performance
- Native routing
- Better throughput
- Low latency
- Simplified networking

---

# 4. What is ENI

## ENI Full Form

ENI =
Elastic Network Interface

---

## Purpose of ENI

ENI acts as:

- Virtual network card attached to EC2

Every worker node can have:

- One primary ENI
- Multiple secondary ENIs

---

## Example

```text
EC2 Node
   |
   |---- ENI-1
   |---- ENI-2
   |---- ENI-3
```

---

# 5. Primary IP vs Secondary IP

## Primary IP

Primary IP is used for:

- Node registration
- SSH
- Kubernetes node communication

### Example

```text
Primary IP:
10.0.1.10
```

---

## Secondary IPs

Secondary IPs are attached to ENIs.

Pods receive these secondary IPs.

### Example

```text
Secondary IPs:
10.0.1.11
10.0.1.12
10.0.1.13
```

---

## Important Point

| IP Type | Purpose |
|---|---|
| Primary IP | Node communication |
| Secondary IP | Pod communication |

---

# 6. How Pods Get IP Addresses

## Step-by-Step Flow

### Step 1

EC2 worker node starts.

---

### Step 2

aws-node DaemonSet starts on node.

---

### Step 3

CNI plugin checks:

- Existing ENIs
- Available secondary IPs

---

### Step 4

If IPs are insufficient:

- New ENI gets attached
- Secondary IPs allocated

---

### Step 5

When Pod starts:

- One secondary IP assigned to Pod

---

## Example

```text
Node Primary IP:
10.0.1.10

ENI:
eni-12345

Secondary IPs:
10.0.1.11
10.0.1.12
10.0.1.13

Pods:
Pod-A -> 10.0.1.11
Pod-B -> 10.0.1.12
Pod-C -> 10.0.1.13
```

---

## Important Point

Pods DO NOT get IPs from:

- Separate pod CIDR

Pods use:

- Secondary IPs attached to ENIs

---

# 7. Pod Communication Flow

## Same Node Pod Communication

```text
Pod-A -> Pod-B
10.0.1.11 -> 10.0.1.12
```

Traffic stays inside node networking.

---

## Cross Node Pod Communication

```text
Node-1 Pod:
10.0.1.11

Node-2 Pod:
10.0.2.15
```

Traffic flows through:

- AWS VPC route tables

No overlay tunnel required.

---

## Pod to Service Communication

```text
Pod
 ↓
kube-proxy
 ↓
Service
 ↓
Destination Pod
```

---

## Pod to Node Communication

```text
Pod -> Node
10.0.1.11 -> 10.0.1.10
```

Direct communication possible.

---

# 8. Node Communication

## Node Registration

Worker nodes register to cluster using:

- Primary private IP

### Example

```text
Node IP:
10.0.1.10
```

---

## Node-to-Node Communication

Traffic uses:

- VPC internal routing

---

# 9. Internet Communication

## Private Subnet Flow

```text
Pod
 ↓
ENI
 ↓
Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

## Public Subnet Flow

```text
Pod
 ↓
ENI
 ↓
Internet Gateway
 ↓
Internet
```

---

# 10. Cross Node Pod Communication

## Example

```text
Pod-A:
10.0.1.11

Pod-B:
10.0.2.15
```

---

## Traffic Flow

```text
Pod-A
 ↓
Node-1 ENI
 ↓
VPC Route Table
 ↓
Node-2 ENI
 ↓
Pod-B
```

---

## Important Point

EKS networking uses:

- Native AWS routing
- Native VPC networking
- No VXLAN overlay

---

# 11. IP Assignment Flow

## Pod Creation Flow

```text
Pod Created
    ↓
Kubelet Requests IP
    ↓
CNI Plugin Checks Available IPs
    ↓
Assign Secondary IP
    ↓
Pod Starts
```

---

## If IPs are unavailable

```text
CNI Plugin
    ↓
Create New ENI
    ↓
Attach ENI to Node
    ↓
Allocate Secondary IPs
```

---

# 12. ENI Limits

Each EC2 instance type has limits.

---

## Example: t3.medium

| Resource | Limit |
|---|---|
| Maximum ENIs | 3 |
| IPs per ENI | 6 |

---

## Example: m5.large

| Resource | Limit |
|---|---|
| Maximum ENIs | 3 |
| IPs per ENI | 10 |

---

## Important Point

Maximum Pods depend on:

- ENI count
- Secondary IP count

---

# 13. Maximum Pods Calculation

## Formula

```text
(Max ENIs × IPs per ENI) - Reserved IPs
```

---

## Example

```text
3 ENIs × 10 IPs = 30

Reserved:
- Primary IPs
- System usage

Approx Pods:
29
```

---

## Check Max Pods

```bash
kubectl describe node <node-name>
```

---

# 14. IP Exhaustion

## What is IP Exhaustion?

When no free IPs remain for Pods.

---

## Symptoms

- Pods remain Pending
- FailedScheduling errors
- Insufficient IP errors

---

## Causes

- Small subnet CIDR
- Too many Pods
- ENI limits reached

---

## Solutions

- Larger subnet
- Bigger EC2 instances
- Add worker nodes
- Enable prefix delegation

---

# 15. Prefix Delegation

## What is Prefix Delegation?

Instead of assigning individual IPs:

- AWS allocates IP prefixes

This improves:

- Faster scaling
- Better IP utilization

---

## Benefits

- More Pods per node
- Faster pod startup
- Better scalability

---

## Enable Prefix Delegation

```bash
kubectl set env daemonset aws-node \
-n kube-system ENABLE_PREFIX_DELEGATION=true
```

---

# 16. Security Groups

Security Groups control traffic.

---

## Pod Communication Depends On

- Node Security Group
- VPC Rules
- Route Tables

---

## Example Rules

| Source | Destination | Port |
|---|---|---|
| Node SG | Node SG | All traffic |
| ALB SG | NodePort | 30000-32767 |

---

# 17. Route Tables

Route tables control packet routing.

---

## Example

```text
10.0.0.0/16 -> local
0.0.0.0/0 -> NAT Gateway
```

---

## Purpose

Used for:

- Pod communication
- Internet access
- Cross subnet communication

---

# 18. NAT Gateway Flow

## Private Subnet Internet Access

```text
Pod
 ↓
Worker Node
 ↓
Private Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

## Important Point

Private subnet Pods require:

- NAT Gateway

for outbound internet access.

---

# 19. AWS VPC CNI Components

| Component | Purpose |
|---|---|
| aws-node | CNI DaemonSet |
| ipamd | IP management |
| kubelet | Pod lifecycle |
| ENI | Network interface |
| Secondary IP | Pod IP |
| Route Table | Routing |
| Security Group | Firewall |

---

# 20. Useful Commands

## View aws-node Pods

```bash
kubectl get pods -n kube-system
```

---

## View Nodes

```bash
kubectl get nodes -o wide
```

---

## View Pod IPs

```bash
kubectl get pods -o wide
```

---

## Describe Node

```bash
kubectl describe node <node-name>
```

---

## View ENIs

```bash
aws ec2 describe-network-interfaces
```

---

## View EC2 Details

```bash
aws ec2 describe-instances
```

---

## View aws-node Logs

```bash
kubectl logs -n kube-system -l k8s-app=aws-node
```

---

# 21. Architecture Diagram

```text
                   Internet
                       |
               Internet Gateway
                       |
                 NAT Gateway
                       |
                Route Table
                       |
          --------------------------------
          |                              |
      Worker Node-1                 Worker Node-2
      Primary IP                    Primary IP
      10.0.1.10                     10.0.2.10
          |                              |
        ENI                            ENI
          |                              |
   Secondary IPs                 Secondary IPs
   10.0.1.11                     10.0.2.11
   10.0.1.12                     10.0.2.12
          |                              |
       Pods                           Pods
```

# 22. Summary

- EKS uses AWS VPC CNI Plugin
- Pods receive secondary VPC IPs
- ENIs provide Pod networking
- aws-node manages IP allocation
- Native VPC routing handles traffic
- Primary IP = Node communication
- Secondary IP = Pod communication
- ENI limits affect maximum Pods
- Prefix Delegation improves scalability
