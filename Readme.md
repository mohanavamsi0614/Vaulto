Alright. Here’s your **PR-ready `README.md` content** for

# [Learning Concept-3] CI/CD Pipeline Execution Model & Responsibility Boundaries

Clean. Structured. Clear separation thinking. This is how a production engineer would explain it. 🚀

---

# CI/CD Pipeline Execution Model & Responsibility Boundaries

## Overview

This document explains:

* How CI and CD pipelines execute
* Where builds, tests, artifacts, and deployments happen
* How responsibilities are divided between application code, pipelines, and infrastructure
* Why separation of responsibilities protects production systems

Modern DevOps systems work because responsibilities are clearly defined and enforced.

---

# 1️⃣ CI vs CD Responsibilities

## Continuous Integration (CI)

CI is responsible for **validating code changes before merge**.

CI answers:

> “Is this code safe to integrate?”

### CI Responsibilities

* Checkout source code
* Run unit tests
* Run linting/static checks
* Build artifacts (e.g., Docker image)
* Tag images
* Push images to registry
* Fail fast if validation fails

### What CI Should NOT Do

* Deploy directly to production
* Modify live infrastructure
* Restart Kubernetes workloads
* Bypass artifact creation

CI builds confidence — it does not release software.

---

## Continuous Deployment (CD)

CD is responsible for **deploying validated artifacts safely to environments**.

CD answers:

> “How do we safely run this version?”

### CD Responsibilities

* Pull pre-built Docker image
* Update Kubernetes manifests
* Apply deployment changes
* Manage rollout strategy
* Handle rollback if needed

### What CD Should NOT Do

* Rebuild source code
* Re-run unit tests
* Modify application logic
* Generate new artifacts

CD moves artifacts — it does not create them.

---

# 2️⃣ Pipeline Execution Flow (Where Things Happen)

Understanding ownership prevents production accidents.

| Action                    | Responsibility   |
| ------------------------- | ---------------- |
| Writing business logic    | Application Code |
| Writing unit tests        | Application Code |
| Running tests             | CI Pipeline      |
| Building Docker image     | CI Pipeline      |
| Tagging image             | CI Pipeline      |
| Pushing image to registry | CI Pipeline      |
| Deploying to Kubernetes   | CD Pipeline      |
| Updating manifests        | CD Pipeline      |
| Restarting failed Pods    | Kubernetes       |
| Rescheduling crashed Pods | Kubernetes       |

Key idea:

* Code defines behavior
* CI validates and packages
* CD deploys artifacts
* Kubernetes runs and heals workloads

Pipelines orchestrate. Infrastructure executes.

---

# 3️⃣ Responsibility Boundaries (Why Separation Matters)

Mixing application logic, pipeline logic, and deployment logic in a single step is dangerous.

### 1. Blast Radius

If CI directly deploys code:

* A broken test step could impact production.
* A build misconfiguration could take down live systems.

Separation limits damage.

---

### 2. PR Review Safety

When responsibilities are separated:

* Code changes are reviewed independently.
* Pipeline changes are reviewed carefully.
* Deployment logic is isolated.

If everything is mixed:

* Reviewers cannot predict downstream impact.
* Hidden deployment behavior may exist.

---

### 3. Predictability

Artifact-based deployment guarantees:

Code → Docker Image → Deployment

If CI builds an image and CD deploys that exact image:

* What was tested is exactly what runs.

If code is deployed directly:

* Environment drift
* Untracked differences
* Harder rollbacks

---

### 4. Rollback Reliability

With artifact-based deployment:

* CD can redeploy a previous image tag.
* ReplicaSets track revision history.
* Rollback is deterministic.

If code is rebuilt during deployment:

* The rebuilt version may not match the original.
* Rollback becomes unreliable.

Clear boundaries enable safe recovery.

---

# 4️⃣ Pipeline Change Impact Analysis

You must reason about impact before merging pipeline changes.

---

## Modifying Test Steps in CI

Impact:

* May allow broken code to merge
* May falsely fail valid builds
* Directly affects validation quality

If test coverage decreases → production risk increases.

---

## Modifying Build/Image Creation Steps

Impact:

* Changes artifact structure
* May introduce runtime failures
* Could break container startup

This affects everything downstream because CD deploys the produced artifact.

---

## Modifying Deployment Steps in CD

Impact:

* Changes rollout behavior
* May affect production traffic
* Could cause downtime
* Could disable rollback logic

Deployment logic has direct production impact. It requires careful review.

---

# 5️⃣ CI/CD Execution Model Diagram

## Execution Flow

![Image](https://cdn.sanity.io/images/lofvu8al/production/e37ce13c88889f048aa2b1acae7d6cbfeea5678f-2048x876.png)

![Image](https://miro.medium.com/0%2AneovUAYgPR1UlMl4.png)

![Image](https://media2.dev.to/dynamic/image/width%3D800%2Cheight%3D%2Cfit%3Dscale-down%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxolzvw5562ewy0a26emj.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2AUx9raDEMqqyWrgz3lMppQA.jpeg)

### Logical Flow Representation

```
Code Change
   ↓
CI Pipeline (Tests + Build)
   ↓
Docker Image (Artifact)
   ↓
CD Pipeline (Deploy)
   ↓
Kubernetes / Cloud Infrastructure
```

Export a clean version as PNG and attach it in your PR.

---

# 6️⃣ Reflection

## Why Should Pipelines Orchestrate Instead of Replace Logic?

Pipelines should orchestrate work — not replace application or infrastructure logic.

Because:

* Application code owns business behavior.
* CI owns validation and artifact creation.
* CD owns deployment.
* Infrastructure owns execution and self-healing.

If pipelines replace infrastructure logic:

* They become tightly coupled.
* Failures become unpredictable.
* Ownership becomes unclear.

Clear ownership enables:

* Safer automation
* Smaller blast radius
* Easier debugging
* Predictable rollbacks
* Maintainable systems

Automation without responsibility boundaries creates chaos.

---
