# Continuous Delivery with ArgoCD and GKE Project

## Overview

This repository contains the infrastructure code for deploying an example voting application using ArgoCD with Google Kubernetes Engine (GKE). The purpose of this project is to demonstrate advanced knowledge and practical implementation of Continuous Delivery (CD) in a Kubernetes environment.

The voting application is a multi-tier system consisting of a frontend voting app and supporting services such as Redis and PostgreSQL. Docker images for the application components are hosted in my personal Docker Hub repository and pulled during the deployment process.

![10k645iekn2c-CICDwithArgoCD](https://github.com/p-georgiadis/vote-deploy/assets/75082315/d44213a4-3673-485c-91e4-ba4139d1fb8a)
## Table of Contents

1. [Getting Started](#getting-started)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Setup](#setup)
5. [Deploying with ArgoCD](#deploying-with-argocd)
6. [Accessing the Application](#accessing-the-application)
7. [Cleaning Up](#cleaning-up)
8. [Project Structure](#project-structure)
9. [References](#references)

## Getting Started

This section provides a quick guide to get the voting application up and running using ArgoCD and GKE. Ensure all prerequisites are met and follow the setup instructions carefully.
![zriylblszxeg-Autosync](https://github.com/p-georgiadis/vote-deploy/assets/75082315/5ca6210d-9799-44de-abcd-e4fed11ab9c3)

## Architecture

The voting application consists of the following components:

- **Voting App**: A Python web application allowing users to vote between two options.
- **Result App**: A Node.js web application displaying the voting results in real-time.
- **Redis**: A queue collecting new votes.
- **Worker**: A .NET service consuming votes from the Redis queue and storing them in the database.
- **PostgreSQL**: A database storing the votes.

The architecture diagram is as follows:

```plaintext
   +-------------+      +-----------+      +---------------+
   |             |      |           |      |               |
   |  Voting App +----->+  Redis    +----->+  Worker        |
   |  (Python)   |      |  Queue    |      |  (.NET)        |
   +------+------+      +------+----+      +------+--------+
          |                    |                   |
          |                    |                   |
          v                    v                   |
   +------+----+         +-----+-----+             |
   |           |         |           |             |
   |  Frontend |         |  Backend  |             |
   |  (Python) |         |  (Node.js)|             |
   +------+----+         +------+----+             |
          |                    |                   |
          +--------+    +------+    +--------------+
                   |    |           |
                   v    v           v
                 +---------+   +---------+
                 |  Redis  |   |PostgreSQL|
                 +---------+   +---------+
```
## Prerequisites

Before deploying the application, ensure you have the following:

1. **Google Cloud SDK**: Installed and authenticated.
2. **kubectl**: Installed and configured to interact with your GKE cluster.
3. **ArgoCD CLI**: Installed for managing the CD pipeline.
4. **Docker**: Installed for building and pushing images.
5. **Docker Hub Account**: With the necessary repositories for the application images.

## Setup

### 1. Google Kubernetes Engine (GKE)

Create a GKE cluster using the Google Cloud Console or CLI:

```sh
gcloud container clusters create voting-app-cluster --zone us-central1-a
```

Configure kubectl to use the GKE cluster:

```sh
gcloud container clusters get-credentials voting-app-cluster --zone us-central1-a
```

### 2. ArgoCD

Install ArgoCD in your Kubernetes cluster:

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access the ArgoCD UI and log in:

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Visit [https://localhost:8080](https://localhost:8080) and log in using the initial admin password retrieved with:

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Deploying with ArgoCD

### Create Application Configuration:

Define the ArgoCD application configuration in a YAML file (`argo-app.yaml`):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: voting-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/yourusername/yourrepo.git'
    path: '.'
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Apply the ArgoCD Application:

```sh
kubectl apply -f argo-app.yaml
```

### Monitor Deployment:

Use the ArgoCD UI or CLI to monitor the deployment status:

```sh
argocd app list
argocd app get voting-app
```

## Accessing the Application

Once deployed, the voting application can be accessed through the GKE external IPs:

- **Voting App**: Accessible on the external IP assigned to the `vote` service on port `31000`.
- **Results App**: Accessible on the external IP assigned to the `result` service on port `31001`.

Retrieve the external IPs with:

```sh
kubectl get services
```

## Cleaning Up

To clean up the resources created in your GKE cluster:

### Remove the ArgoCD application:

```sh
argocd app delete voting-app
```

### Delete the GKE cluster:

```sh
gcloud container clusters delete voting-app-cluster --zone us-central1-a
```

## Project Structure

```plaintext
.
├── argo-app.yaml
├── vote-ui-deployment.yaml
├── vote-ui-svc.yaml
└── README.md
```

## References

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Google Kubernetes Engine (GKE) Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
