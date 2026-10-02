# Architecture

## Overview

The Todo Summary Assistant uses a CI/CD and GitOps-based deployment workflow.

```text
Developer
   |
   v
GitHub Repository
   |
   v
Jenkins CI
   |
   +--> Maven build and tests
   |
   +--> Docker image build
   |
   +--> Push images to Docker Hub
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
Kubernetes Cluster
   |
   +----------------------+
   |                      |
   v                      v
Frontend Service      Backend Service
   |                      |
   v                      v
Frontend Pods          Backend Pods
                          |
                          v
                    MySQL StatefulSet
                          |
                          v
                    Persistent Volume

Components
GitHub
The Git repository stores application source code, Dockerfiles, Jenkins pipeline configuration, Kubernetes manifests, GitOps configuration, and operational documentation.
Jenkins
Jenkins performs continuous integration:
1. Checks out the source code.
2. Runs backend Maven tests.
3. Builds the backend Docker image.
4. Builds the frontend Docker image.
5. Tags the images using the Jenkins build number.
6. Pushes the images to Docker Hub.
Jenkins does not directly deploy workloads to Kubernetes.
Docker Hub
Docker Hub stores the versioned backend and frontend container images produced by Jenkins.
Argo CD
Argo CD provides the GitOps deployment mechanism.
It watches the Git repository containing the Kubernetes desired state and synchronizes that state with the Kubernetes cluster.
Automated synchronization is enabled with pruning and self-healing.
Kubernetes
The application runs in the todo-prod namespace.
The workload consists of:
- Frontend Deployment with 2 replicas
- Backend Deployment with 2 replicas
- MySQL StatefulSet with 1 replica
- Frontend ClusterIP Service
- Backend ClusterIP Service
- MySQL headless Service
- MySQL PersistentVolumeClaim
- ConfigMap for non-sensitive backend configuration
- Kubernetes Secret for sensitive configuration
Request Flow
Browser
   |
   v
Frontend Service
   |
   v
Frontend Pod / Nginx
   |
   | /api/*
   v
Backend Service
   |
   v
Backend Pod
   |
   v
MySQL Service
   |
   v
MySQL Pod
   |
   v
Persistent Storage

The frontend Nginx server serves the React application and proxies /api/ requests to the backend Kubernetes Service.
Configuration and Secrets
Non-sensitive configuration is stored in a Kubernetes ConfigMap.
Sensitive values such as database passwords, API keys, and webhook values are stored in Kubernetes Secrets and are excluded from Git tracking.
GitOps Flow
The deployment flow is:
1. A change is pushed to GitHub.
2. Jenkins builds and tests the application.
3. Jenkins builds versioned container images.
4. Jenkins pushes the images to Docker Hub.
5. The desired image version is stored in GitOps configuration.
6. Argo CD detects the Git change.
7. Argo CD synchronizes Kubernetes with the desired state.
8. Kubernetes performs the rollout.
CI is therefore separated from deployment.
Rollback
Rollback is performed through Git.
If a faulty image or configuration is committed to the GitOps configuration:
1. Argo CD detects the change.
2. Kubernetes attempts to apply the new desired state.
3. The faulty deployment can be reverted using Git.
4. Argo CD detects the reverted commit.
5. Kubernetes is synchronized back to the previous working state.
This keeps Git as the source of truth for the deployed state.
Production Considerations
The local Kubernetes environment uses Kind for development and demonstration.
For a production AWS deployment, the Kubernetes workloads can be moved to Amazon EKS.
For production, the database should preferably use a managed highly available database service rather than a single MySQL pod with local cluster storage.
Additional production considerations include:
- TLS termination
- Ingress or load balancer
- Centralized logging
- Metrics and alerting
- Image vulnerability scanning
- Network policies
- External secret management
- Automated backups
- High availability
  EOF
