## Overview

This document explains the complete lifecycle of an application artifact — from a Git commit to running containers inside a Kubernetes cluster.

---

# 1️⃣ Artifact Flow Explanation

## Step 1: Git Commit / PR Merge → CI Trigger

When a developer pushes code or merges a Pull Request into the `main` branch, a webhook event triggers the CI pipeline.

Example GitHub Actions trigger:

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
```

The CI system automatically starts the workflow. No manual deployment is required.

---

## Step 2: CI Pipeline Builds Docker Image

Inside the CI pipeline:

1. Repository source code is checked out.
2. Dependencies are installed.
3. Automated tests are executed.
4. If tests pass, a Docker image is built.

Example:

```bash
docker build -t my-app:latest .
```

The `Dockerfile` defines:

* Base runtime image
* Application dependencies
* Build steps
* Startup command

This process creates an immutable Docker image containing:

* Application code
* Runtime
* Dependencies
* Configuration

---

## Step 3: Image Tagging & Digest

During the build process, the image is tagged.

Common strategies:

* `latest`
* Git commit SHA
* Semantic version (e.g., v1.0.0)

Example:

```bash
docker tag my-app my-registry/my-app:${GITHUB_SHA}
```

Each image also gets a **digest** (SHA256 hash).

* **Tag** → Human-readable label
* **Digest** → Cryptographic identity of the image contents

Tags can move. Digests are immutable.

---

## Step 4: Push to Container Registry

After tagging, CI authenticates and pushes the image to a container registry.

Examples:

* Docker Hub
* GitHub Container Registry
* Amazon Elastic Container Registry

Push command:

```bash
docker push my-registry/my-app:${GITHUB_SHA}
```

The registry acts as a centralized artifact store

---

## Step 5: Kubernetes Deployment

Kubernetes references the image in a Deployment manifest:

```yaml
containers:
  - name: my-app
    image: my-registry/my-app:abc123
```

When applied:

```bash
kubectl apply -f deployment.yaml
```

Kubernetes performs the following:

1. Schedules a Pod.
2. The node pulls the image from the registry.
3. A container is created from the image.
4. The application starts running.

If replicas = 3 → Kubernetes runs 3 identical containers from the same immutable image.

---

# 2️⃣ End-to-End Artifact Flow Diagram

## Architecture Flow

![Image](https://miro.medium.com/v2/resize%3Afit%3A2000/1%2ACH2R5552IjZCTqhgaBpXHw.jpeg)

![Image](https://miro.medium.com/0%2A3_uIz_YMiyZxMwKn)

![Image](https://creately.com/static/assets/guides/kubernetes-architecture-diagram/it-and-engineering-kubernetes-architecture-diagram.svg)

![Image](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)

## Linear Representation

```
Git Commit / PR Merge
        ↓
CI Pipeline (Build + Test)
        ↓
Docker Image (Tagged + Digest)
        ↓
Container Registry
        ↓
Kubernetes Deployment
        ↓
Running Containers (Pods)
```

Export your own clean diagram (PNG) and attach it to the PR.

Keep it minimal and clearly labeled.

---

# 3️⃣ Reflection

## Why is deploying immutable Docker images safer than deploying code directly?

Deploying immutable artifacts improves:

### Reliability

The exact same image tested in CI is what runs in production.

### Rollbacks

Previous image versions can be redeployed instantly without rebuilding.

### Traceability

Each running container maps to:
Container → Image Tag → Image Digest → Git Commit

This enables precise debugging and auditing.

### Consistency Across Environments

Development, staging, and production all run the same artifact.

Direct code deployment introduces:

* Dependency drift
* Environment inconsistencies
* Hard-to-debug runtime changes

Immutable Docker images eliminate these risks.
