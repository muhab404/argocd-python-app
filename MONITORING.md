# Monitoring Stack

Simple monitoring solution for Python Flask application using Prometheus and Grafana.

## Components

- **Prometheus**: Metrics collection (7 days retention, 5Gi storage)
- **Grafana**: Visualization (admin/admin login, 2Gi storage)
- **AlertManager**: Basic alerting
- **Node Exporter**: System metrics
- **kube-state-metrics**: Kubernetes metrics

## Python App Changes

Added `prometheus-flask-exporter` to requirements.txt for HTTP metrics:
- Automatic `/metrics` endpoint
- Request count, duration, and status metrics

## Deployment

Deploys via ArgoCD using kube-prometheus-stack Helm chart:
- Namespace: `monitoring` (auto-created)
- Chart: `prometheus-community/kube-prometheus-stack:45.0.0`

## Access

### Grafana
```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```
- URL: http://localhost:3000
- Login: admin/admin

### Prometheus
```bash
kubectl port-forward svc/monitoring-prometheus -n monitoring 9090:9090
```
- URL: http://localhost:9090

## Metrics Available

- HTTP request metrics from Flask app
- Kubernetes cluster metrics
- Node system metrics
- Pod and deployment status

## Files

- `monitoring-values.yaml`: Helm configuration
- `apps/monitoring.yaml`: ArgoCD application