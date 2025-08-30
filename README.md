# ETCD Operations - Argo CD Deployment

This repository contains Kubernetes manifests for deploying an ETCD cluster using Argo CD with the Application of Applications pattern.

## Repository Structure

```
├── .github/workflows/
│   └── deploy-argocd.yml          # GitHub Actions workflow for deployment
├── manifests/
│   └── etcd/
│       ├── kustomization.yaml     # Kustomize configuration
│       ├── app-etcd.yaml          # Argo CD Application for ETCD
│       ├── statefulset.yaml       # ETCD StatefulSet
│       ├── service.yaml           # ETCD Service
│       ├── ingress.yaml           # ETCD Ingress
│       ├── pdb.yaml               # Pod Disruption Budget
│       ├── networkpolicy.yaml     # Network Policy
│       └── *.yaml                 # Certificate and PKI manifests
└── root-app.yaml                  # Root Argo CD Application
```

## Deployment Architecture

This setup uses the **Application of Applications** pattern:

1. **Root Application** (`root-app.yaml`): Manages all child applications
2. **Child Applications** (`manifests/etcd/app-etcd.yaml`): Manages specific workloads

## Prerequisites

- Kubernetes cluster with Argo CD installed
- GitHub repository secrets configured:
  - `PSYS_CENTOS_1_KUBE_CONFIG`: Base64 encoded kubeconfig file
- Argo CD namespace (`argocd`) must exist
- Proper RBAC permissions for Argo CD

## GitHub Workflow Features

The workflow includes:

### 🔍 **Validation Stage**
- Validates Kustomize builds
- Dry-run validation of Kubernetes manifests
- Syntax validation of Argo CD applications

### 🚀 **Deployment Stage**
- Secure kubeconfig setup with proper permissions
- Argo CD installation verification
- Root application deployment
- Application status monitoring
- Comprehensive status reporting

### 🧹 **Cleanup Stage**
- Automatic cleanup on deployment failures
- Resource cleanup to prevent orphaned resources

## Kustomize Integration

The ETCD application is configured to use Kustomize for manifest management:

```yaml
source:
  repoURL: https://github.com/padminisys/etcd-ops.git
  targetRevision: main
  path: manifests/etcd
  kustomize:
    buildOptions: "--enable-helm"
```

## Deployment Process

### Automatic Deployment
The workflow triggers automatically on:
- Push to `main` branch (when manifests change)
- Manual workflow dispatch

### Manual Deployment
```bash
# Apply root application directly
kubectl apply -f root-app.yaml

# Monitor application status
kubectl get applications -n argocd -w
```

## Monitoring and Troubleshooting

### Check Application Status
```bash
# List all applications
kubectl get applications -n argocd

# Get detailed status
kubectl describe application etcd-ops-root -n argocd
kubectl describe application etcd-cluster -n argocd
```

### Check ETCD Resources
```bash
# Check ETCD pods
kubectl get pods -n default -l app.kubernetes.io/name=etcd

# Check ETCD services
kubectl get svc -n default -l app.kubernetes.io/name=etcd

# Check ETCD logs
kubectl logs -n default -l app.kubernetes.io/name=etcd
```

### Common Issues

1. **Argo CD Not Installed**
   - Ensure Argo CD is properly installed in the cluster
   - Verify `argocd` namespace exists

2. **Sync Issues**
   - Check repository access permissions
   - Verify branch and path configurations
   - Review Argo CD server logs

3. **Kustomize Build Failures**
   - Validate kustomization.yaml syntax
   - Ensure all referenced resources exist
   - Check for circular dependencies

## Security Considerations

- Kubeconfig is stored as a GitHub secret and decoded securely
- Proper file permissions (600) are set for kubeconfig
- Cleanup procedures prevent resource leakage
- Network policies restrict ETCD access

## Sync Policy

Applications are configured with:
- **Automated sync**: Enabled with prune and self-heal
- **Retry policy**: 5 attempts with exponential backoff
- **Sync options**: Automatic namespace creation

## Contributing

1. Make changes to manifests in the `manifests/etcd/` directory
2. Update `kustomization.yaml` if adding new resources
3. Test changes locally with `kustomize build manifests/etcd`
4. Push to `main` branch to trigger deployment

## Support

For issues related to:
- **Kubernetes manifests**: Check the `manifests/etcd/` directory
- **Argo CD configuration**: Review `app-etcd.yaml` and `root-app.yaml`
- **Deployment pipeline**: Check `.github/workflows/deploy-argocd.yml`