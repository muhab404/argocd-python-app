# ArgoCD Python App Configuration

GitOps configuration for deploying the Python Flask application and its dependencies using ArgoCD with the App of Apps pattern.

## Architecture

This repository implements the **App of Apps** pattern where a root application manages multiple child applications:

```
root-app (ArgoCD Application)
├── python-app (Helm Chart)
├── nginx-ingress (Helm Chart)
└── monitoring (Prometheus Stack)
```

## Applications

### Root Application (`root-app.yaml`)
- **Purpose**: Manages all child applications
- **Source**: This repository (`apps/` directory)
- **Sync Policy**: Automated with prune and self-heal
- **Namespace**: `argocd`

### Python App (`apps/python-app.yaml`)
- **Purpose**: Deploys the Python Flask API
- **Source**: `https://github.com/muhab404/helm-python-app`
- **Type**: Helm chart
- **Namespace**: `python-app`
- **Features**:
  - Automated sync with prune and self-heal
  - Uses values from Helm repository
  - Image tags updated by CI/CD pipeline

### NGINX Ingress (`apps/nginx-ingress.yaml`)
- **Purpose**: Provides ingress controller for external access
- **Source**: Official NGINX Ingress Helm chart
- **Chart Version**: 4.10.1
- **Namespace**: `ingress-nginx` (auto-created)
- **Configuration**:
  - LoadBalancer service type
  - Ingress class: `nginx`
  - Controller value: `k8s.io/ingress-nginx`

### Monitoring (`apps/monitoring.yaml`)
- **Purpose**: Deploys Prometheus monitoring stack
- **Source**: Prometheus Community Helm charts
- **Chart**: `kube-prometheus-stack` v45.0.0
- **Namespace**: `monitoring`
- **Includes**: Prometheus, Grafana, AlertManager

## Prerequisites

- ArgoCD installed in Kubernetes cluster
- Access to the target Kubernetes cluster
- GitHub repository access for source code

## Installation

### 1. Install ArgoCD (if not already installed)

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Deploy Root Application

```bash
kubectl apply -f root-app.yaml
```

### 3. Access ArgoCD UI

```bash
# Port forward to ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Access: `https://localhost:8080`
- Username: `admin`
- Password: (from command above)

## GitOps Workflow

1. **Code Changes**: Developer pushes code to Python app repository
2. **CI/CD Pipeline**: 
   - Builds and tests application
   - Creates Docker image
   - Pushes to ECR
   - Updates Helm chart values with new image tag
3. **ArgoCD Sync**: 
   - Detects changes in Helm repository
   - Automatically syncs new image to Kubernetes
   - Ensures desired state matches Git state

## Application Structure

```
argocd-python-app/
├── root-app.yaml           # Root ArgoCD application
└── apps/
    ├── python-app.yaml     # Python Flask app
    ├── nginx-ingress.yaml  # Ingress controller
    └── monitoring.yaml     # Prometheus stack
```

## Sync Policies

All applications use automated sync with:
- **Prune**: Remove resources not in Git
- **Self-Heal**: Automatically fix drift from desired state
- **Create Namespace**: Auto-create target namespaces

## Accessing Applications

### Python App
```bash
# Via ingress (after setting up DNS/hosts)
curl http://python-app.local/users

# Via port forward
kubectl port-forward svc/python-app -n python-app 5000:5000
curl http://localhost:5000/users
```

### Monitoring (Grafana)
```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```
Access: `http://localhost:3000`

### ArgoCD Applications Status
```bash
# List all applications
kubectl get applications -n argocd

# Check specific application
kubectl describe application python-app -n argocd
```

## Troubleshooting

### Application Not Syncing
```bash
# Check application status
kubectl get app -n argocd

# View application details
kubectl describe app python-app -n argocd

# Force sync via CLI
argocd app sync python-app
```

### Image Pull Issues
- Ensure ECR credentials are configured
- Check `imagePullSecrets` in Helm values
- Verify image repository and tag

### Ingress Issues
- Confirm NGINX ingress controller is running
- Check ingress resource configuration
- Verify DNS/hosts file setup

## Security Considerations

- Applications run in separate namespaces for isolation
- RBAC policies control ArgoCD access
- Image scanning in CI/CD pipeline
- Secrets managed through Kubernetes secrets
- Network policies can be added for additional security

## Monitoring and Observability

The monitoring stack provides:
- **Prometheus**: Metrics collection
- **Grafana**: Visualization dashboards
- **AlertManager**: Alert routing and management
- **Node Exporter**: System metrics
- **kube-state-metrics**: Kubernetes object metrics

## Customization

To customize deployments:
1. Fork the Helm chart repository
2. Modify values in the Helm repository
3. Update the `repoURL` in application manifests
4. ArgoCD will automatically sync changes