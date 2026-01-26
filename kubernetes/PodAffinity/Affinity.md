# Kubernetes Pod Affinity & Pod Anti-Affinity – Real-Time Examples

This documentation explains **Pod Affinity** and **Pod Anti-Affinity** in Kubernetes using **real-world production scenarios**.

---

## 📌 What is Pod Affinity?

**Pod Affinity** tells Kubernetes:

> ✅ “Schedule this pod close to another specific pod.”

### Why?
- Reduce network latency
- Improve performance
- Keep tightly coupled services together

---

## 📌 What is Pod Anti-Affinity?

**Pod Anti-Affinity** tells Kubernetes:

> ❌ “Do NOT schedule this pod on the same node as another specific pod.”

### Why?
- Avoid resource contention
- Improve application stability
- Prevent one pod from affecting another

---

## 🎯 Real-Time Scenario 1: Pod Anti-Affinity (CPU-Heavy Pod)

### 🔥 Problem Statement

In real production systems:

- Some pods **consume high CPU**
- They can **slow down other pods** on the same node
- This causes:
    - Increased response time
    - Performance degradation
    - Application instability

### 🧠 Requirement

> If there is a **CPU-intensive pod**,  
> **do not schedule other application pods on the same node**.

This is a **perfect use case for Pod Anti-Affinity**.

---

