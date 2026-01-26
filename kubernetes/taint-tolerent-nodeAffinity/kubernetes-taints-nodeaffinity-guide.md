# Kubernetes Taints, Tolerations, NodeSelector and Node Affinity
A Practical Real-World Guide

---

## Introduction

In this guide, we will understand how Kubernetes decides where a Pod should run and how we can control that behavior using:

- Taints
- Tolerations
- NodeSelector
- Node Affinity

This document is written based on a real-world production-like scenario.

---

## Scenario

We have a Kubernetes cluster with two types of nodes:

| Node Type | Description |
|---------|------------|
| Normal Node | High CPU and Memory |
| Dummy Node | Low CPU and Memory |

We run two applications:

| Application | Resource Requirement |
|-----------|---------------------|
| Java App | High CPU & Memory |
| Python App | Low CPU & Memory |

Requirements:

- Java app must never run on Dummy Node.
- Python app should run only on Dummy Node.
- Java app should use Normal Node.
- Python app should not consume Normal Node resources.

---

## 1. Taints

Taint is applied on a Node.  
It tells Kubernetes that the node has a limitation.

Meaning:
Only pods that tolerate this taint are allowed.

Command:
```
kubectl taint nodes <node-name> power=Dummy:NoExecute
```
Example:
```bash
kubectl taint nodes desktop-worker2 power=Dummy:NoExecute
```
Meaning:

- Node has a taint: power=Dummy
- Only pods with matching toleration can stay
- Others will not be scheduled or will be evicted

---

## 2. Taint Effects

Taints support three effects:

| Effect | Meaning |
|------|-------|
| NoSchedule | Pods without toleration will not be scheduled |
| PreferNoSchedule | Try not to schedule but allow if needed |
| NoExecute | Do not allow and evict existing pods |

For strict separation, we use:
`
NoExecute
`
---

## 3. Tolerations

Tolerations are applied on Pods.  
It means the pod can tolerate the taint.

Python app toleration:
```
tolerations:
- key: "power"
  operator: "Equal"
  value: "Dummy"
  effect: "NoSchedule"
```
Now Python pods can run on Dummy Node.  
Java pods cannot.

---

## 4. Problem

Taints only block pods.  
They do not force pods to go to a specific node.

Python pods can still go to Normal Node.

So we need Node selection.

---

## 5. Node Labels

Label the Dummy Node:

```bash 
kubectl label node desktop-worker2 cpu=Dummy
```


Now the node has:
`
cpu=Dummy
`
---

## 6. NodeSelector

In Pod spec:
```
nodeSelector:
  cpu: Dummy
```
Meaning:
Schedule this pod only on nodes with cpu=Dummy.

Limitation:
Only one value per key is allowed.

---

## 7. Node Affinity

Node Affinity is an advanced version of NodeSelector.

Example:
```bash
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: cpu
          operator: In
          values:
          - Dummy
          - VeryDummy
```


Meaning:
Pod can run on any node with:
```
- cpu=Dummy
- cpu=VeryDummy
```
---

## 8. Required vs Preferred

| Type | Meaning |
|-----|-------|
| requiredDuringScheduling | Mandatory |
| preferredDuringScheduling | Best effort |

---

## 9. IgnoredDuringExecution

Once a pod is running, Kubernetes will not kill it even if labels change.

---

## Final Result

| Application | Runs On |
|-----------|--------|
| Java App | Normal Node |
| Python App | Dummy Node |

Perfect workload separation achieved.

---

## Summary

| Feature | Purpose |
|------|--------|
| Taint | Blocks pods |
| Toleration | Allows pods |
| NodeSelector | Simple node binding |
| NodeAffinity | Advanced node binding |
