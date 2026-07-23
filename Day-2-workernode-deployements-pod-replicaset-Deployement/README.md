# Day 2: Worker Node Deployments

This directory contains examples of Kubernetes workload resources that run on worker nodes: Pods, ReplicaSets, and Deployments.

## 📋 Overview

- **Pod**: The smallest deployable unit in Kubernetes
- **ReplicaSet**: Ensures a specified number of pod replicas are running
- **Deployment**: Higher-level abstraction that manages ReplicaSets for declarative updates

## 🔍 Detailed YAML Explanations

### 2.1 Pod (pod.yml)

**Purpose**: Creates a single container running nginx web server

**Key Components**:
- `apiVersion: v1` - Core Kubernetes API for basic resources
- `kind: Pod` - Resource type for individual containers
- `metadata.name` - Unique identifier for the pod
- `metadata.labels` - Key-value pairs for organization and selection
- `spec.containers` - List of containers to run in the pod
- `image: nginx:latest` - Docker image to use (nginx web server)
- `ports.containerPort: 80` - Port the container listens on internally

**YAML Structure**:
```yaml
apiVersion: v1          # API version for Pod resources
kind: Pod              # Resource type
metadata:              # Pod identification and labels
  name: nginx-pod
  labels:
    app: nginx
spec:                  # Pod specification
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 2.2 ReplicaSet (replicaset.yml)

**Purpose**: Ensures exactly 3 nginx pods are always running

**Key Components**:
- `apiVersion: apps/v1` - Apps API for workload controllers
- `kind: ReplicaSet` - Resource type for pod replication
- `spec.replicas: 3` - Desired number of pod instances
- `spec.selector.matchLabels` - Labels to identify managed pods
- `spec.template` - Pod template for creating new pods
- Inherits all pod specification from the template

**YAML Structure**:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3                    # Number of replicas
  selector:                      # Pod selector
    matchLabels:
      app: nginx
  template:                      # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 2.3 Deployment (Deployemnt.yml)

**Purpose**: Provides declarative pod management with update capabilities

**Key Components**:
- Same API version and structure as ReplicaSet
- `kind: Deployment` - Higher-level resource type
- Manages ReplicaSets internally for updates
- Supports rolling updates, rollbacks, and pause/resume
- `containerPort: 81` - Modified port for demonstration

**YAML Structure**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 81      # Modified port
```

## ⚠️ Drawbacks and Limitations

### Pod Drawbacks

**1. No Self-Healing**
- If a pod crashes, it stays terminated
- Manual intervention required to restart
- No automatic replacement of failed pods

**2. No Scaling**
- Cannot automatically scale based on load
- Manual creation/deletion of pod instances
- No load balancing across multiple pods

**3. No Updates/Rollbacks**
- No declarative update mechanism
- Updating requires manual pod replacement
- No rollback capability for failed updates

**4. Limited Lifecycle Management**
- No health checks or readiness probes by default
- No resource limits/requests management
- Manual cleanup required

**5. Not Recommended for Production**
- Should not be used directly in production
- Better to use higher-level controllers (Deployments)

### ReplicaSet Drawbacks

**1. No Declarative Updates**
- Cannot perform rolling updates
- Updating requires manual ReplicaSet replacement
- No built-in rollback mechanism

**2. Complex Update Process**
- To update pod template, must create new ReplicaSet
- Old ReplicaSet must be manually deleted
- Risk of downtime during updates

**3. Limited Update Control**
- No control over update speed or strategy
- All pods update simultaneously (not rolling)
- No pause/resume capability

**4. Manual Scaling Only**
- Scaling requires explicit commands
- No auto-scaling based on metrics
- No integration with HPA (Horizontal Pod Autoscaler)

**5. Selector Limitations**
- Cannot manage pods with different labels
- Selector is immutable after creation
- Cannot adopt existing pods easily

## ✅ Why Use Deployments Instead?

**Deployments solve ReplicaSet limitations**:

1. **Declarative Updates**: `kubectl set image` triggers rolling updates
2. **Rollback Support**: `kubectl rollout undo` for instant rollback
3. **Update Strategies**: Rolling updates, blue-green, canary deployments
4. **Pause/Resume**: Pause updates for debugging, resume when ready
5. **Version History**: Track deployment changes and rollbacks
6. **Integration**: Works with auto-scaling and advanced scheduling

## 🛠️ Practical Commands

### 2.1 Basic Pod Deployment

```bash
# Apply the pod configuration
kubectl apply -f pod.yml

# Check pod status
kubectl get pods

# View detailed pod information
kubectl describe pod nginx-pod

# Access pod logs
kubectl logs nginx-pod

# Execute commands inside pod
kubectl exec -it nginx-pod -- /bin/bash

# Clean up
kubectl delete -f pod.yml
```

**What you'll learn**:
- Pod specification syntax
- Container configuration
- Port exposure basics
- Pod lifecycle management

### 2.2 ReplicaSet for Replication

```bash
# Apply ReplicaSet
kubectl apply -f replicaset.yml

# Check ReplicaSet and pods
kubectl get rs
kubectl get pods

# Scale the ReplicaSet
kubectl scale rs nginx-replicaset --replicas=5

# View ReplicaSet details
kubectl describe rs nginx-replicaset

# Clean up
kubectl delete -f replicaset.yml
```

**What you'll learn**:
- Pod replication management
- Label selectors
- Automatic pod replacement
- Scaling concepts

### 2.3 Deployment for Updates

```bash
# Apply Deployment
kubectl apply -f Deployemnt.yml

# Check deployment status
kubectl get deployments
kubectl get pods

# Update the deployment (change image version)
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.21

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# View deployment history
kubectl rollout history deployment/nginx-deployment

# Rollback if needed
kubectl rollout undo deployment/nginx-deployment

# Clean up
kubectl delete -f Deployemnt.yml
```

**What you'll learn**:
- Declarative pod management
- Rolling updates and rollbacks
- Deployment strategies
- Version management

## 🔄 Evolution: Pod → ReplicaSet → Deployment

1. **Pod**: Basic container wrapper (don't use directly)
2. **ReplicaSet**: Adds replication (don't use directly)
3. **Deployment**: Adds updates and rollbacks (use this!)

**Best Practice**: Always use Deployments for stateless applications. Use StatefulSets for stateful applications.

## 📊 Comparison Table

| Feature       | Pod | ReplicaSet | Deployment |
|---------|-----|------------|------------|
| Self-healing | ❌ | ✅      | ✅ |
| Scaling | ❌ | ✅           | ✅ |
| Updates | ❌ | ❌           | ✅ |
| Rollbacks | ❌ | ❌         | ✅ |
| Production Ready | ❌ | ⚠️  | ✅ |

## 🎯 Key Takeaways

- **Pods** are the atomic unit but shouldn't be used directly
- **ReplicaSets** provide replication but lack update management
- **Deployments** are the recommended way to run applications
- Understanding this hierarchy helps choose the right resource type
- Each layer builds upon the previous, adding more capabilities
