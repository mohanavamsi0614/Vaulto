

## Overview

This document explains how Kubernetes manages the lifecycle of an application — from deployment creation to pod scheduling, health management, rollouts, failures, and self-healing.

Kubernetes operates using a **desired state model**, continuously reconciling actual cluster state with declared configuration.

---

# 1️⃣ Deployment → ReplicaSet → Pods (Lifecycle Core)

## Deployment: Defining Desired State

A **Deployment** defines the desired state of an application:

* Container image
* Number of replicas
* Resource requirements
* Health probes
* Update strategy

Example:

```yaml
replicas: 3
image: my-app:v2
```

This means:

> “There must always be 3 healthy Pods running this image.”

Kubernetes does not directly create Pods from a Deployment.
Instead, it creates a ReplicaSet.

---

## ReplicaSet: Maintaining Desired State

When a Deployment is applied:

1. Kubernetes creates a **ReplicaSet**
2. The ReplicaSet creates the specified number of Pods
3. It continuously monitors pod count

If:

* A Pod crashes → ReplicaSet creates a new one
* A Node fails → Pods are recreated on another node
* Replicas increase → new Pods are created
* Replicas decrease → Pods are terminated

ReplicaSets are the **self-healing enforcement mechanism** behind Deployments.

---

## Pod Creation & Scheduling

After the ReplicaSet creates a Pod:

1. The Pod enters `Pending`
2. The Scheduler selects a suitable Node based on:

   * CPU & memory requests
   * Node availability
   * Taints & tolerations
3. The kubelet on the chosen Node:

   * Pulls the container image
   * Starts the container
   * Reports status back to the API server

Once the container starts and passes health checks, the Pod becomes `Running`.

---

## Desired State vs Current State (Reconciliation)

Kubernetes continuously compares:

* Desired state (from Deployment spec)
* Current state (actual Pods running)

If they differ, Kubernetes reconciles automatically.

Example:
Desired = 3 Pods
Current = 2 Pods

Action → Create 1 new Pod

Kubernetes guarantees **state correction**, not application correctness.

---

# 2️⃣ Deployment & Rolling Update Mechanics

## Rolling Update Strategy (Default)

When a new image version is deployed:

1. A new ReplicaSet is created.
2. New Pods are started with the new image.
3. Old Pods are gradually terminated.
4. Traffic shifts incrementally.
5. Availability is maintained.

Kubernetes ensures:

* Some old Pods remain running
* New Pods must become Ready before scaling down old ones

---

## Successful Rollout

A rollout is successful when:

* All new Pods are `Ready`
* Old ReplicaSet scales down to zero
* Deployment status shows updated replicas = desired replicas

---

## Failed Rollout

Rollout may fail if:

* New Pods fail readiness probes
* Containers crash (CrashLoopBackOff)
* Resource constraints prevent scheduling

In this case:

* Old ReplicaSet remains available
* Deployment can be rolled back
* Kubernetes halts further scaling

ReplicaSets track revision history, enabling rollback.

---

# 3️⃣ Health Probes & Resource Configuration

## Liveness Probe

Checks if the container is alive.

If it fails:

* Container is restarted.

Misconfiguration → infinite restarts → CrashLoopBackOff.

---

## Readiness Probe

Checks if the Pod is ready to receive traffic.

If it fails:

* Pod is removed from Service endpoints.
* Traffic is not routed to it.

Incorrect readiness probes often cause:

* Deployment appearing stuck
* Partial outages

---

## Startup Probe

Used for slow-starting applications.

Prevents liveness checks from killing containers prematurely during startup.

---

## CPU & Memory Requests

Requests determine:

* Scheduling eligibility

If requests are too high:

* Pod remains `Pending`
* Scheduler cannot place it

---

## CPU & Memory Limits

Limits control runtime behavior.

* CPU limit exceeded → throttling
* Memory limit exceeded → Pod is `OOMKilled`

Incorrect limits frequently cause production instability.

---

# 4️⃣ Common Pod States & Failure Conditions

## Pending

**Meaning:** Pod not scheduled yet
**Causes:**

* Insufficient resources
* Unsatisfiable node constraints
  **Kubernetes Response:** Keeps retrying scheduling

---

## CrashLoopBackOff

**Meaning:** Container repeatedly crashes
**Causes:**

* Application bug
* Misconfigured environment variables
* Failing liveness probe
  **Kubernetes Response:** Restarts container with exponential backoff

---

## ImagePullBackOff

**Meaning:** Image cannot be pulled
**Causes:**

* Wrong image name
* Authentication failure
* Registry unavailable
  **Kubernetes Response:** Retries image pull

---

## OOMKilled

**Meaning:** Container exceeded memory limit
**Cause:** Memory usage > defined limit
**Kubernetes Response:** Terminates container and restarts

---

# 5️⃣ Kubernetes Lifecycle Diagram

## Application Flow

![Image](https://substackcdn.com/image/fetch/%24s_%214KwS%21%2Cf_auto%2Cq_auto%3Agood%2Cfl_progressive%3Asteep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcc0a7875-ee23-4906-adc7-b3d6b47ced0e_2252x2752.png)

![Image](https://iximiuz.com/kubernetes-vs-age-old-infra-patterns/kubernetes-service-min.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AV3grVwrlokIKTH2rvoY-lg.png)

![Image](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates2.svg)

### Logical Flow Representation

```
Deployment
   ↓
ReplicaSet
   ↓
Pod Creation
   ↓
Scheduling
   ↓
Container Start
   ↓
Health Checks
   ↓
Running / Restart / Reschedule
```

Export a clean labeled diagram as PNG and include it in the PR.

---

# 6️⃣ Reflection

## Why Does Kubernetes Maintain Desired State Instead of Guaranteeing Application Correctness?

Kubernetes focuses on maintaining desired state because:

* It is a platform, not an application debugger.
* It ensures infrastructure-level reliability.
* It automates recovery through self-healing.
* It continuously reconciles state without manual intervention.

Kubernetes guarantees:

* Correct number of replicas
* Pod restarts on failure
* Rescheduling on node failure
* Traffic routing based on readiness

It does NOT guarantee:

* Your application logic is correct
* Your code won’t crash
* Your memory usage is optimal

Platform responsibility ends at infrastructure correctness.
Application correctness is the developer’s responsibility.

This separation enables scalable automation and reliability.
