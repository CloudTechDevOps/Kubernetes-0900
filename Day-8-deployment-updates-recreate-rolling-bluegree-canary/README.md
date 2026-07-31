# Kubernetes Deployment Strategies

# Overview

This project explains different Kubernetes deployment strategies used to release new versions of applications.

Deployment strategies help DevOps teams:

* Release applications safely
* Reduce downtime
* Minimize production risk
* Perform controlled rollouts
* Rollback quickly during failures

---

# Deployment Strategies Covered

1. Recreate Deployment
2. Rolling Update Deployment
3. Blue-Green Deployment
4. Canary Deployment

---

# Why Deployment Strategies are Important?

In production environments:

* Applications must remain available
* Downtime should be minimized
* New releases may contain bugs
* Rollbacks should be quick
* User impact should be controlled

Deployment strategies solve these challenges.

---

# Kubernetes Default Strategy

By default, Kubernetes uses:

```text
RollingUpdate
```

This ensures applications are updated gradually with minimal downtime.

---

# 1. Recreate Deployment Strategy

# What is Recreate Strategy?

In recreate deployment strategy:

* Old application version is completely terminated
* New application version is deployed afterward

Workflow:

```text
Old Pods Deleted → New Pods Created
```

---

# Architecture

Version-1 Pods → Deleted → Version-2 Pods Created

---

# How Recreate Works

Step 1:

Kubernetes stops all existing pods.

Step 2:

Kubernetes deploys new version pods.

Step 3:

Application becomes available again.

---

# Advantages

* Simple deployment process
* Easy to understand
* No version compatibility issues
* Only one application version runs at a time

---

# Disadvantages

* Causes downtime
* Users cannot access application during deployment
* Risky for production applications

---

# Best Use Cases

Suitable for:

* Development environments
* Internal applications
* Non-critical applications
* Applications requiring database schema changes

---

# Kubernetes Configuration

```yaml
strategy:
  type: Recreate
```

---

# Recreate Deployment Flow

1. User accesses version 1
2. Deployment starts
3. Old pods terminate
4. Downtime occurs
5. New pods start
6. Users access version 2

---

# Real World Example

Suppose:

* Application version 1 is running
* New version 2 is ready

Kubernetes:

* Deletes all version 1 pods
* Starts version 2 pods

During this period application becomes unavailable.

---

# 2. Rolling Update Deployment Strategy

# What is Rolling Update?

Rolling update gradually replaces old pods with new pods.

Workflow:

```text
One Old Pod Removed → One New Pod Added
```

This process continues until all pods are updated.

---

# Architecture

Version-1 Pods + Version-2 Pods Running Together Temporarily

---

# How Rolling Update Works

Step 1:

One old pod terminates.

Step 2:

One new pod starts.

Step 3:

Traffic shifts gradually.

Step 4:

Process repeats until deployment completes.

---

# Advantages

* Zero downtime
* Safer deployment
* Easy rollback
* Default Kubernetes strategy
* Continuous application availability

---

# Disadvantages

* Multiple versions run simultaneously
* Possible compatibility issues
* Longer deployment process

---

# Best Use Cases

Suitable for:

* Web applications
* APIs
* Production workloads
* High availability systems

---

# Kubernetes Configuration

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

---

# Understanding maxSurge

Defines:

Extra pods Kubernetes can create during update.

Example:

```yaml
maxSurge: 1
```

Kubernetes creates one extra pod during deployment.

---

# Understanding maxUnavailable

Defines:

How many pods can be unavailable during update.

Example:

```yaml
maxUnavailable: 1
```

Only one pod can be unavailable at a time.

---

# Rolling Update Flow

Version 1:

```text
Pod-1 Pod-2 Pod-3
```

Deployment starts:

```text
Pod-1 deleted
Pod-4 created
```

Then:

```text
Pod-2 deleted
Pod-5 created
```

Until all pods become latest version.

---

# Real World Example

Suppose:

* 10 replicas running
* Deploying new version

Kubernetes updates pods gradually:

```text
1 → 2 → 3 → 4...
```

Users experience no downtime.

---

# 3. Blue-Green Deployment Strategy

# What is Blue-Green Deployment?

Blue-Green deployment uses:

* Two identical environments
* One active environment
* One inactive environment

Terminology:

* Blue = Current production version
* Green = New version

---

# Architecture

Users → Service → Blue Environment

Green environment remains idle until testing completes.

After validation:

Traffic switches from Blue to Green.

---

# How Blue-Green Works

Step 1:

Blue environment serves production traffic.

Step 2:

Deploy new version to Green environment.

Step 3:

Test Green environment.

Step 4:

Switch traffic from Blue to Green.

Step 5:

Green becomes production.

---

# Advantages

* Near zero downtime
* Instant rollback
* Safer deployments
* Easy testing before release

---

# Disadvantages

* Higher infrastructure cost
* Requires double resources
* Complex setup

---

# Best Use Cases

Suitable for:

* Production critical applications
* Banking systems
* E-commerce platforms
* High availability systems

---

# Blue-Green Deployment Flow

Initial:

```text
Users → Blue Environment
```

Testing:

```text
Users → Blue
Testing → Green
```

After switch:

```text
Users → Green
```

---

# Traffic Switching

Traffic switching usually happens through:

* Kubernetes Service
* Load Balancer
* Ingress Controller
* Service Mesh

---

# Rollback Process

If Green fails:

```text
Switch traffic back to Blue
```

Rollback becomes instant.

---

# Kubernetes Implementation

Usually implemented using:

* Two deployments
* One service
* Label switching

Example:

Blue:

```yaml
labels:
  version: blue
```

Green:

```yaml
labels:
  version: green
```

Service switches selector.

---

# Real World Example

Suppose:

* Blue version running in production
* Green version deployed separately

After successful testing:

Service redirects traffic to Green.

---

# 4. Canary Deployment Strategy

# What is Canary Deployment?

Canary deployment releases application gradually to a small percentage of users first.

Example:

* 5% traffic → New version
* 95% traffic → Old version

If successful:

Traffic gradually increases.

---

# Architecture

Users → Load Balancer

Traffic Split:

* Majority → Stable version
* Small percentage → Canary version

---

# How Canary Works

Step 1:

Deploy canary version.

Step 2:

Route small traffic percentage.

Step 3:

Monitor:

* Errors
* CPU
* Latency
* Logs

Step 4:

Increase traffic gradually.

Step 5:

Promote canary to production.

---

# Advantages

* Lowest deployment risk
* Real user testing
* Easy rollback
* Controlled rollout
* Better monitoring

---

# Disadvantages

* Complex implementation
* Requires traffic management
* Requires monitoring tools

---

# Best Use Cases

Suitable for:

* Large scale applications
* SaaS platforms
* Critical APIs
* Production applications

---

# Canary Deployment Flow

Initial:

```text
100% → Version 1
```

Canary release:

```text
95% → Version 1
5% → Version 2
```

Expansion:

```text
50% → Version 1
50% → Version 2
```

Final:

```text
100% → Version 2
```

---

# Traffic Control Methods

Canary deployments often use:

* Istio
* Linkerd
* NGINX Ingress
* AWS Load Balancer
* Service Mesh

---

# Rollback Process

If issues detected:

```text
Route all traffic back to stable version
```

Rollback becomes fast.

---

# Real World Example

Suppose:

Netflix releases new recommendation engine.

Deployment:

* 5% users receive new version
* Metrics monitored carefully
* Gradually rollout to all users

---

# Deployment Strategy Comparison

| Strategy       | Downtime | Complexity | Rollback | Cost   | Risk     |
| -------------- | -------- | ---------- | -------- | ------ | -------- |
| Recreate       | High     | Low        | Slow     | Low    | High     |
| Rolling Update | Low      | Medium     | Medium   | Low    | Medium   |
| Blue-Green     | Very Low | Medium     | Fast     | High   | Low      |
| Canary         | Very Low | High       | Fast     | Medium | Very Low |

---

# Strategy Selection Guide

# Use Recreate When

* Downtime acceptable
* Internal tools
* Small applications

---

# Use Rolling Update When

* Zero downtime required
* Standard production apps
* Kubernetes default behavior preferred

---

# Use Blue-Green When

* Fast rollback needed
* Production critical systems
* Infrastructure budget available

---

# Use Canary When

* Need gradual rollout
* Large user base
* Advanced monitoring available
* High reliability required

---

# Monitoring Deployment Strategies

Monitoring tools used:

* Prometheus
* Grafana
* Datadog
* New Relic
* ELK Stack

Important metrics:

* Error rates
* CPU usage
* Memory usage
* Request latency
* Pod restarts

---

# Kubernetes Deployment Commands

# Create Deployment

```bash
kubectl apply -f deployment.yaml
```

---

# Check Deployment

```bash
kubectl get deploy
```

---

# Check Pods

```bash
kubectl get pods
```

---

# Deployment Rollout Status

```bash
kubectl rollout status deployment APP_NAME
```

---

# Rollback Deployment

```bash
kubectl rollout undo deployment APP_NAME
```

---

# Deployment History

```bash
kubectl rollout history deployment APP_NAME
```

---

# Best Practices

## Use Readiness Probes

Ensure traffic only reaches healthy pods.

---

## Configure Resource Limits

Prevent resource exhaustion.

---

## Use Monitoring Tools

Track deployment health.

---

## Automate Rollbacks

Quick recovery during failures.

---

## Test in Staging First

Validate deployments before production.

---

# Production Recommendations

## Small Applications

Use:

```text
Rolling Update
```

---

## Enterprise Applications

Use:

```text
Blue-Green or Canary
```

---

## High Traffic Applications

Use:

```text
Canary Deployment
```

---

# CI/CD Integration

Deployment strategies are commonly integrated with:

* Jenkins
* GitHub Actions
* GitLab CI
* Argo CD
* Spinnaker
* Tekton

---

# DevOps Workflow Example

Developer Push → CI Pipeline → Docker Build → Kubernetes Deployment → Monitoring → Rollback if Failure

---

# Conclusion

Kubernetes deployment strategies help organizations deploy applications safely and efficiently.

Each strategy has different:

* Risk levels
* Downtime characteristics
* Complexity
* Infrastructure requirements

Choosing the correct deployment strategy depends on:

* Application criticality
* User traffic
* Infrastructure budget
* Rollback requirements
* Monitoring capabilities

Understanding deployment strategies is essential for modern DevOps and cloud-native application delivery.
