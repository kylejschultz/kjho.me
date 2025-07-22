# GitOps Workflow

This homelab is managed using GitOps principles with Flux CD, ensuring all infrastructure and applications are declaratively defined and automatically deployed from Git.

## Overview

**GitOps Philosophy:**

- Git is the single source of truth for all cluster configuration
- All changes flow through Git commits and pull requests
- Flux continuously reconciles cluster state with Git repository
- Manual `kubectl` commands are avoided in favor of declarative manifests

**Key Components:**

- **Flux CD**: GitOps operator managing deployments
- **Helm**: Package manager for applications
- **Kustomize**: Configuration management and patching
- **1Password Connect**: Secret synchronization from vault

## Repository Structure

```
k8s/
├── orchestration/           # Flux and foundational resources
│   ├── flux-system/         # Flux controllers and configuration
│   │   ├── gotk-components.yaml    # Flux toolkit components
│   │   ├── gotk-sync.yaml          # Git repository sync
│   │   └── flux-notifications/     # Discord webhook alerts
│   └── foundational/        # Base cluster resources
│       ├── namespaces/      # Namespace definitions
│       └── helmrepos/       # Helm repository sources
└── core/                    # Application deployments
    ├── database/           # PostgreSQL and Redis
    ├── network/            # Networking components
    ├── observability/      # Monitoring applications
    ├── security/           # Authentication services
    └── storage/            # Storage systems
```

## Flux Configuration

### GitRepository Source

Flux monitors this Git repository for changes and applies them to the cluster:

📋 [View Git Sync Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/orchestration/flux-system/gotk-sync.yaml)

**Key Settings:**

- **Repository**: https://github.com/kylejschultz/kjho.me
- **Branch**: `main`
- **Interval**: 1 minute reconciliation
- **Path**: `k8s/orchestration`

### Kustomization Hierarchy

Flux applies resources in a structured hierarchy:

1. **Foundational**: Namespaces, Helm repositories
2. **Core Infrastructure**: Storage, networking, databases
3. **Applications**: Security, observability, services

📋 [View Orchestration Kustomization](https://github.com/kylejschultz/kjho.me/blob/main/k8s/orchestration/kustomization.yaml)

## Deployment Process

### Standard GitOps Flow

1. **Code Changes**: Modify YAML manifests or add new resources
2. **Commit & Push**: Changes pushed to `main` branch
3. **Flux Sync**: Flux detects changes within 1 minute
4. **Reconciliation**: Flux applies changes to cluster
5. **Notifications**: Discord alerts on success/failure

### HelmRelease Workflow

Applications are deployed using HelmReleases:

1. **Helm Repository**: Define chart source in `helmrepos/`
2. **Namespace**: Create namespace in `namespaces/`
3. **HelmRelease**: Define application configuration
4. **Secrets**: 1Password items for sensitive values
5. **Dependencies**: Use `dependsOn` for deployment ordering

Example HelmRelease structure:
```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: app-name
  namespace: app-namespace
spec:
  interval: 5m
  chart:
    spec:
      chart: chart-name
      version: "1.0.0"
      sourceRef:
        kind: HelmRepository
        name: chart-repo
  values:
    # Application configuration
```

## Secrets Management

### 1Password Integration

Secrets are stored in 1Password vault and synchronized to Kubernetes:

**OnePasswordItem Resources:**

- Define secret mappings from 1Password vault
- Sync credentials, API keys, certificates
- Automatic secret rotation and updates

📋 [Example 1Password Item](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/authentik-onepassword-item.yaml)

**Secret References in HelmReleases:**
```yaml
env:
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: db_password
```

## Monitoring and Alerts

### Discord Notifications

Flux sends deployment notifications to Discord:

- **Success**: Green notifications for successful deployments
- **Failures**: Red alerts with error details
- **Reconciliation**: Status updates on configuration changes

📋 [View Discord Alerts Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/orchestration/flux-system/flux-notifications/discord-alerts.yaml)

### Useful Commands

```bash
# Check Flux system status
flux get kustomizations

# View HelmRelease status
flux get helmreleases --all-namespaces

# Force reconciliation
flux reconcile kustomization flux-system

# Check Git repository sync
flux get sources git

# View Flux logs
kubectl logs -n flux-system -l app=source-controller

# Suspend/resume reconciliation
flux suspend kustomization core-database
flux resume kustomization core-database
```

## Best Practices

### Development Workflow

**For Infrastructure Changes:**
1. Create feature branch for testing
2. Test changes in development environment
3. Create pull request for review
4. Merge to `main` for automatic deployment

**For Application Updates:**
1. Update chart version or values in HelmRelease
2. Commit changes to trigger deployment
3. Monitor deployment via Discord notifications
4. Rollback via Git revert if needed

### Configuration Management

- **Minimal Manifests**: Keep deployments simple and focused
- **Individual Files**: Separate each resource into its own file
- **Consistent Naming**: Use consistent resource naming patterns
- **Documentation**: Link to source files in documentation
- **Dependencies**: Use explicit `dependsOn` relationships

### Security Considerations

- **No Secrets in Git**: All sensitive data in 1Password vault
- **Least Privilege**: Minimal RBAC permissions for Flux
- **Network Policies**: Restrict pod-to-pod communication
- **Image Scanning**: Use trusted container images

---

📁 **Related Files:**
- [Flux System Kustomization](https://github.com/kylejschultz/kjho.me/blob/main/k8s/orchestration/flux-system/kustomization.yaml)
- [Core Kustomization](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/kustomization.yaml)
- [Foundational Resources](https://github.com/kylejschultz/kjho.me/tree/main/k8s/orchestration/foundational)
