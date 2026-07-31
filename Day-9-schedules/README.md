# Kubernetes Scheduling Concepts

This repository explains important Kubernetes scheduling concepts:

- DaemonSet
- Node Selector
- Node Affinity Preferred
- Node Affinity Required
- Taints and Tolerations

---

# 1. DaemonSet

## What is DaemonSet?

A DaemonSet ensures that a specific Pod runs on all nodes in a Kubernetes cluster.

Whenever a new node joins the cluster, Kubernetes automatically creates the Pod on that node.

---

## Use Cases

- Monitoring Agents
- Log Collection
- Security Agents
- Networking Plugins

Examples:
- Fluentd
- Prometheus Node Exporter
- Calico

---

## Architecture

```text
            Kubernetes Cluster
    --------------------------------
    Node-1  ---> Monitoring Pod
    Node-2  ---> Monitoring Pod
    Node-3  ---> Monitoring Pod
```

---

## DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: nginx-daemonset

spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx
```

---

# 2. Node Selector

## What is Node Selector?

`nodeSelector` is the simplest way to schedule Pods on specific nodes using labels.

---

## Add Label to Node

```bash
kubectl label nodes worker-node env=production
```

---

## Node Selector YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  nodeSelector:
    env: production

  containers:
  - name: nginx
    image: nginx
```

---

## Architecture

```text
Node-1 -> env=production
Node-2 -> env=dev

Pod with nodeSelector env=production
        |
        ---> Scheduled on Node-1
```

---

## Important Points

- Simple scheduling method
- Exact label matching only
- Cannot use complex conditions

---

# 3. Node Affinity

Node Affinity is an advanced version of nodeSelector.

It provides more flexible scheduling rules.

---

# 3.1 Preferred Node Affinity

## What is Preferred Node Affinity?

`preferredDuringSchedulingIgnoredDuringExecution`

This is a soft rule.

Kubernetes tries to place the Pod on matching nodes, but if unavailable, Pod can run on another node.

---

## YAML Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: preferred-affinity-pod

spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: env
            operator: In
            values:
            - production

  containers:
  - name: nginx
    image: nginx
```

---

## Architecture

```text
Preferred Rule:
env=production

Node-1 -> env=production
Node-2 -> env=dev

Kubernetes prefers Node-1
But can use Node-2 if needed
```

---

## Important Points

- Soft scheduling rule
- Pod can run on non-matching nodes
- Used for preferred placement

---

# 3.2 Required Node Affinity

## What is Required Node Affinity?

`requiredDuringSchedulingIgnoredDuringExecution`

This is a hard rule.

Pod runs only on matching nodes.

If no matching node exists, Pod remains in Pending state.

---

## YAML Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: required-affinity-pod

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - production

  containers:
  - name: nginx
    image: nginx
```

---

## Architecture

```text
Required Rule:
env=production

Node-1 -> env=production
Node-2 -> env=dev

Pod ONLY schedules on Node-1
Else Pod stays Pending
```

---

## Important Points

- Hard scheduling rule
- More powerful than nodeSelector
- Supports multiple operators

Operators:
- In
- NotIn
- Exists
- DoesNotExist
- Gt
- Lt

---

# 4. Taints and Tolerations

## What are Taints?

Taints are applied on nodes to repel Pods.

---

## What are Tolerations?

Tolerations are applied on Pods to allow scheduling on tainted nodes.

---

## Real-Time Analogy

```text
Taint       = Restricted Area
Toleration  = Special Permission Pass
```

---

## Add Taint to Node

```bash
kubectl taint nodes worker-node env=production:NoSchedule
```

---

## Toleration YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"

  containers:
  - name: nginx
    image: nginx
```

---

## Taint Effects

| Effect | Description |
|---|---|
| NoSchedule | Pod will not schedule |
| PreferNoSchedule | Kubernetes avoids scheduling |
| NoExecute | Existing Pods also removed |

---

## Architecture

```text
Node-1
Taint:
env=production:NoSchedule

Normal Pod
    ❌ Cannot Schedule

Pod with Toleration
    ✅ Can Schedule
```

---

# Difference Between Concepts

| Concept | Purpose | Type |
|---|---|---|
| nodeSelector | Simple node selection | Hard Rule |
| Preferred Affinity | Preferred scheduling | Soft Rule |
| Required Affinity | Mandatory scheduling | Hard Rule |
| Taints | Restrict Pods from nodes | Node Rule |
| Tolerations | Allow Pods on tainted nodes | Pod Permission |
| DaemonSet | Run Pod on all nodes | Controller |

---

# Conclusion

These Kubernetes scheduling concepts help control:

- Where Pods run
- Which nodes accept Pods
- Infrastructure-level Pod deployment
- Advanced workload management

Understanding these concepts is important for Kubernetes cluster administration and production deployments.
