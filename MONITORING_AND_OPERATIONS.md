# Monitoring and Operations

## Monitoring Goals

The monitoring design should detect application, container, Kubernetes, database, and infrastructure problems before they become major incidents.

## Metrics

Important metrics include:

- Pod CPU and memory usage
- Container restart count
- Pod readiness status
- Deployment replica availability
- HTTP request rate
- HTTP 4xx and 5xx responses
- Application response latency
- Database availability
- Database storage usage
- Kubernetes node CPU and memory
- Kubernetes node readiness
- Persistent Volume capacity

## Logs

Application and container logs can initially be inspected using:

```bash
kubectl logs <pod-name> -n todo-prod

For production, application and infrastructure logs should be collected centrally so logs remain available when individual pods or nodes disappear.
A centralized logging stack such as Elasticsearch, Logstash and Kibana, or a managed logging platform, can be used.
Alerts
Recommended alerts include:
Condition	Example response
Pod not ready	Investigate deployment and pod events
CrashLoopBackOff	Inspect application logs and configuration
High restart count	Investigate crashes or resource limits
High 5xx rate	Investigate application and backend dependencies
High latency	Investigate application and database performance
Node NotReady	Check node health and rescheduling
PVC usage above threshold	Increase storage or clean unused data
Deployment replicas unavailable	Investigate scheduling or application health


Early Detection
Readiness and liveness probes provide an initial health signal.
Resource requests and limits help Kubernetes schedule workloads and prevent uncontrolled resource consumption.
For a production AWS environment, Kubernetes metrics and application metrics can be integrated with Prometheus/Grafana or a managed cloud monitoring service.
Operational Commands
kubectl get pods -n todo-prod
kubectl get deployments -n todo-prod
kubectl get events -n todo-prod --sort-by=.lastTimestamp
kubectl top pods -n todo-prod
kubectl top nodes

Application troubleshooting:
kubectl logs <pod-name> -n todo-prod
kubectl describe pod <pod-name> -n todo-prod

Deployment health:
kubectl rollout status deployment/todo-summary-backend -n todo-prod
kubectl rollout status deployment/todo-summary-frontend -n todo-prod

