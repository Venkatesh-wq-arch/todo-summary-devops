# Failure and Rollback

## Faulty Production Deployment

If a faulty image is promoted through GitOps:

1. Update the image tag in the GitOps manifest.
2. Commit and push the change to Git.
3. Argo CD detects the Git change and synchronizes Kubernetes.
4. Readiness and liveness probes help prevent unhealthy pods from receiving traffic.
5. Check pod status and application health.
6. Revert the faulty Git commit.
7. Argo CD synchronizes the reverted state.
8. Kubernetes recreates the workload using the previous known-good image.

Rollback is performed through Git rather than manually changing the live Kubernetes deployment.

This was tested by deploying a deliberately unavailable backend image tag and then reverting the Git commit.

## Application Crash

If an application container crashes:

- Kubernetes restarts the container when required.
- Liveness probes detect an unhealthy container.
- Deployment replicas provide availability while an individual pod is unhealthy.
- Pod logs and restart counts are used during investigation.

Useful commands:

```bash
kubectl get pods -n todo-prod
kubectl describe pod <pod-name> -n todo-prod
kubectl logs <pod-name> -n todo-prod
Jenkins Failure
If Jenkins becomes unavailable:
- Existing Kubernetes workloads continue running.
- Argo CD continues managing the desired Git state.
- Existing container images remain available in the registry.
- CI resumes after Jenkins is restored.
Jenkins failure therefore does not directly terminate an already running workload.
Secret Exposure
If a secret is accidentally exposed:
1. Revoke or rotate the exposed credential.
2. Create or update the Kubernetes Secret with the replacement credential.
3. Restart affected workloads if required.
4. Remove the secret from the repository.
5. Review Git history and repository access.
6. Investigate whether the credential was accessed.
Secrets are excluded from the Git repository using .gitignore. Kubernetes Secret resources are bootstrapped separately from the GitOps manifests.
Kubernetes Node Failure
If a worker node becomes unavailable:
- Kubernetes detects the node failure.
- Pods managed by Deployments can be rescheduled onto healthy nodes when capacity is available.
- Multiple backend and frontend replicas reduce the impact of a single pod or node failure.
The local Kind environment uses a single MySQL replica with persistent storage. For production, the database should use a highly available managed database service rather than relying on a single Kubernetes database pod.
Recovery Principle
Git is the source of truth for application deployment configuration.
Git Change
    |
    v
Argo CD
    |
    v
Kubernetes
    |
    v
Application

Rollback:
Faulty Git Commit
       |
       v
   git revert
       |
       v
    Argo CD
       |
       v
Known-Good Kubernetes State

