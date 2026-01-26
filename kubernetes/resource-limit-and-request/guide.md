# Kubernetes Resource Requests and Resource Limits

## Overview

In this guide, we will understand **Resource Requests** and **Resource Limits** in Kubernetes, why they are important, and how they solve real-world production issues.

Resource management is one of the most important best practices when running applications in a Kubernetes cluster.

---

## Real-World Scenario

Assume we have **two applications** running in a Kubernetes cluster:

- A **Spring Boot (Java) application**
- A **Python application**

Both applications are deployed and working fine initially.

Suddenly, the **Python application becomes very slow**.

After investigation, we find that:
- The Python pod is **not getting enough CPU and memory**
- The **Spring Boot application is consuming unusually high CPU and memory**
- This happens due to a **memory leak or unexpected load** in the Java application

Because of this:
- The Java application is consuming most of the node resources
- Other pods (Python app) are starving for CPU and memory

---

## Another Problem: Scheduler Behavior

While this issue is already happening, a **new Python application** is deployed.

What does Kubernetes Scheduler do?

- It schedules the **new pod on the same node**
- It **does not check actual runtime resource usage**
- By default, Kubernetes assumes the pod needs **very little resources**

So now we have **two problems**:

1. One application is consuming excessive CPU and memory
2. Scheduler is placing new pods on nodes without enough resources

---

## Solution: Resource Requests and Resource Limits

Kubernetes solves these problems using:

- **Resource Requests**
- **Resource Limits**

---

## What is Resource Limit?

**Resource Limit** defines the **maximum amount of CPU and memory** a pod is allowed to use.

### Behavior:
- If a pod exceeds its **CPU limit**, it will be throttled
- If a pod exceeds its **memory limit**, Kubernetes will kill the pod (OOMKilled) and restart it

### Benefits:
- Prevents one pod from consuming all node resources
- Protects other applications from performance issues
- Helps control memory leaks and runaway processes

---

## What is Resource Request?

**Resource Request** defines the **minimum CPU and memory** required by a pod.

### Behavior:
- Kubernetes Scheduler checks resource requests **before scheduling**
- Pod will be scheduled **only on nodes that have enough free resources**
- If sufficient resources are not available, the pod remains in **Pending state**

### Benefits:
- Guarantees minimum resources for your application
- Prevents pods from being scheduled on overloaded nodes
- Improves application stability

---

## Why Both Are Important

| Feature | Purpose |
|------|--------|
| Resource Request | Scheduling decision (minimum guarantee) |
| Resource Limit | Runtime protection (maximum usage) |

---

## How to Configure Resource Requests and Limits

You can define resource requests and limits in your **Pod or Deployment YAML file**.

### Example: Deployment Resource Configuration

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

---

## Production Best Practice

For **production workloads**:

- Always define resource requests and limits
- Prevent one pod from affecting others
- Improve cluster stability and performance

---

## Conclusion

Resource Requests and Limits help Kubernetes schedule pods intelligently and protect applications from resource starvation.
