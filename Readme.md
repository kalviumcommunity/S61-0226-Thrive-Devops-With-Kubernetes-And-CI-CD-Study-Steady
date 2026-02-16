# Artifact Flow in CI/CD

**Source → Image → Registry → Cluster**

In this learning task, I understood how a **small code change** moves from **Git** to a **running application in Kubernetes** using a CI/CD pipeline.

Earlier, I thought code is deployed directly.
Now I understand that **code is first converted into an artifact (Docker image)** and only that artifact is deployed.

---

## 🔹 What I Learned About Source Code (Git)

* Every change starts with a **Git commit**
* A commit has a **unique commit hash**
* This commit clearly tells **which version of code** is being built

📌 **Learning:**
The commit hash is the identity of the code.

---

## 🔹 What I Learned About CI Pipeline

* CI pipeline starts automatically when code is pushed or merged
* CI does **not deploy code**
* CI:

  * Pulls the code
  * Runs tests
  * Builds a Docker image
  * Tags the image

📌 **Learning:**
CI’s main job is to **create a Docker image artifact**, not deployment.

---

## 🔹 What I Learned About Docker Images

* A Docker image contains:

  * Code
  * Runtime
  * Dependencies
* Once an image is built, it **never changes**
* Any new code change creates a **new image**

Example:

```
Commit A → app:commit-a1b2
Commit B → app:commit-b3c4
```

📌 **Learning:**
Docker images are **immutable**, which makes systems reliable.

---

## 🔹 What I Learned About Container Registry

* Docker images are stored in a **container registry**
* Kubernetes does **not** pull code from Git
* Kubernetes pulls **images from the registry**

📌 **Learning:**

* **Image tags** are human-friendly names
* **Image digests** uniquely identify the exact image
* Registries help in **versioning and traceability**

---

## 🔹 What I Learned About Kubernetes Deployment

* Kubernetes Deployments mention:

  * Image name
  * Image tag
  * Number of replicas
* Kubernetes:

  * Pulls image from registry
  * Runs containers
  * Restarts failed containers
  * Performs rolling updates

📌 **Learning:**
Kubernetes runs **sealed images**, not source code.

---

## 🔹 What I Learned About Rollbacks

* Rollbacks are easy because:

  * Images are immutable
  * Registries store old images
  * Deployment points to a specific image

If a release fails:

* Select the previous image
* Redeploy it
* Kubernetes rolls back safely

📌 **Learning:**
No rebuilding. No guessing. Just redeploy an old image.

---

## 🔹 Complete Flow I Learned

```
Git Commit
   ↓
CI Pipeline
   ↓
Docker Image
   ↓
Container Registry
   ↓
Kubernetes Cluster
```

---

#  Kubernetes Application Lifecycle & Deployment Mechanics

## Overview

This document explains how Kubernetes manages application workloads throughout their lifecycle — from creation and scheduling to updates, failures, and recovery.

It covers:

- Deployment → ReplicaSet → Pods
- Rolling updates
- Health probes
- Resource configuration
- Pod failure states
- Self-healing behavior

---

# 1️ Kubernetes Lifecycle (Deployment → ReplicaSet → Pods)

## Desired State vs Current State

Kubernetes works using a **desired state model**.

When we create a Deployment, we define the desired state of the system.  
For example:

- Run 3 replicas
- Use image version v1
- Maintain high availability

Kubernetes continuously compares the **current state** of the cluster with the **desired state**.

If there is any difference, Kubernetes automatically corrects it.  
This continuous correction process is called **reconciliation**.

---

## Deployment → ReplicaSet → Pods Flow

###  Deployment
- Defines the desired state (replica count, image version, update strategy).
- Manages updates and rollbacks.
- Creates and manages ReplicaSets.

###  ReplicaSet
- Created automatically by the Deployment.
- Ensures the specified number of Pods are always running.
- Recreates Pods if they fail.

###  Pods
- Smallest deployable unit in Kubernetes.
- Contain one or more containers.
- Run the application.

---

## Pod Scheduling Process

1. Deployment creates a ReplicaSet.
2. ReplicaSet creates Pods.
3. The Kubernetes Scheduler:
   - Checks available Nodes.
   - Evaluates CPU & memory availability.
4. Pod is assigned to a suitable Node.
5. The kubelet on that Node starts the container.

---

## What Happens if a Pod Crashes?

- If a container crashes → kubelet restarts it.
- If a Pod fails completely → ReplicaSet creates a new Pod.
- If a Node fails → Pods are rescheduled to other Nodes.

This automatic recovery behavior is called **self-healing**.

---

# 2 Deployment & Rolling Update Mechanics

## How Rolling Updates Work

When we update an application (for example, image v1 → v2):

1. Deployment creates a new ReplicaSet with the new image.
2. New Pods are started gradually.
3. Old Pods are terminated slowly.
4. Availability is maintained using:
   - `maxUnavailable`
   - `maxSurge`

This ensures zero or minimal downtime.

---

## Successful Rollout

A rollout is successful when:

- All new Pods are running and ready.
- Old ReplicaSet is scaled down to zero.
- Application is fully running on the new version.

---

## Failed Rollout

If new Pods fail to start:

- They may enter CrashLoopBackOff.
- Readiness probes may fail.
- Deployment rollout pauses.
- Kubernetes allows rollback to the previous working version.

ReplicaSets track both:
- Old version
- New version

---

# 3️ Health Probes & Resource Configuration

## Health Probes

###  Liveness Probe
- Checks if the container is alive.
- If it fails → container is restarted.

###  Readiness Probe
- Checks if the container is ready to serve traffic.
- If it fails → Pod is removed from Service endpoints.

###  Startup Probe
- Used for slow-starting applications.
- Prevents early restarts during initialization.

---

## CPU & Memory Configuration

### Resource Requests
- Used by the scheduler.
- Determines where the Pod can be placed.
- Ensures sufficient resources are reserved.

### Resource Limits
- Maximum resources a container can use.
- If memory exceeds limit → container is OOMKilled.
- If CPU exceeds limit → CPU throttling occurs.

---

## Impact of Misconfiguration

Incorrect configuration can cause:

- CrashLoopBackOff (due to failed probes)
- OOMKilled (due to low memory limit)
- Pending state (due to high resource requests)
- Unavailable application (due to readiness probe failure)

---

# 4️ Common Pod States & Failure Conditions

##  Pending
**Meaning:** Pod created but not scheduled.  
**Cause:** Insufficient resources or constraints.  
**Response:** Kubernetes waits until resources are available.

---

##  CrashLoopBackOff
**Meaning:** Container keeps crashing repeatedly.  
**Cause:** Application errors or bad configuration.  
**Response:** Kubernetes retries with exponential backoff.

---

##  ImagePullBackOff
**Meaning:** Failed to pull container image.  
**Cause:** Incorrect image name or registry issue.  
**Response:** Kubernetes retries pulling the image.

---

##  OOMKilled
**Meaning:** Container exceeded memory limit.  
**Cause:** Memory limit too low or memory leak.  
**Response:** Kubernetes kills and restarts container.

---

# 5️ Kubernetes Lifecycle Diagram

```

Deployment
↓
ReplicaSet
↓
Pod Creation
↓
Scheduler Assigns Node
↓
Container Start (kubelet)
↓
Health Checks (Probes)
↓
Running State
↙             ↘
Restart        Reschedule

```

Export this diagram as an image and attach it to your PR.

---

# 6️ Reflection

Kubernetes focuses on maintaining the **desired state**, not application correctness.

The platform:

- Automates deployment and scaling
- Provides self-healing
- Maintains infrastructure reliability

However, Kubernetes cannot fix:

- Application logic bugs
- Business logic errors

There is a clear boundary:

- Kubernetes ensures infrastructure health.
- Developers ensure application correctness.

This separation allows Kubernetes to provide reliable automation while keeping responsibility clearly divided.
