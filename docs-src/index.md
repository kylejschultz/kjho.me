<div align="center">
  <img src="https://github.com/kylejschultz/kjho.me/raw/docs/assets/images/logo_transparent.png" alt="KJHome Logo" width="200" style="border: none; box-shadow: none; background: transparent;" />
</div>

# KJHome - Self-Hosted Kubernetes Homelab Docs

A reference guide for managing and maintaining my self-hosted Kubernetes homelab, providing a centralized place for documentation processes, workflows, nuances, and quick reference troubleshooting steps.

## 📋 Documentation Index

- [**Authentik Configuration**](Authentik-Configuration) - Details on authentication flows, policies, and providers
- [**Infrastructure Overview**](Infrastructure-Overview) - Hardware, networking, and core services
- [**GitOps Workflow**](GitOps-Workflow) - Flux deployment patterns and procedures
- [**Service Directory**](Service-Directory) - Configuration for all deployed applications
- [**Troubleshooting**](Troubleshooting) - Common issues and solutions

## 🏗️ Infrastructure Overview

### Hardware
- **Cluster**: 3x Intel NUCs (nuc1, nuc2, nuc3) + 1x Worker VM (nucx)
- **Storage**:
    - **Longhorn**: Distributed block storage
        - *NUCs*: mount portion of local disks
        - *Worker VM*: mount large SSD volumes from host

### Core Technologies
- **GitOps**: Flux CD for all deployments
- **Package Management**: Helm charts for applications
- **Secrets Management**: 1Password Operator
- **Authentication**: Authentik with Discord OAuth
- **Monitoring**: Discord webhooks, UptimeKuma
- **Networking**: Cloudflare Tunnels for secure ingress

### Key Services
- **PostgreSQL & Redis**: Shared databases
- **Authentik**: Identity provider
- **UptimeKuma**: Service monitoring
- **Longhorn**: Distributed storage

## 🔗 Quick Access URLs

- **Authentik**: https://auth.kjho.me
- **UptimeKuma**: https://uptime.kjho.me
- **Longhorn**: https://longhorn.kjho.me

## 🚀 Getting Started

For recreating or understanding this setup:

1. Review the [Infrastructure Overview](infrastructure/gitops-workflow.md) for architecture insights
2. Understand [GitOps Workflow](infrastructure/gitops-workflow.md) for deployment methods
3. Examine [Authentik Configuration](services/security/authentik-setup.md) for authentication setup
4. Refer to the [Service Directory](infrastructure/service-directory.md) for application configurations

### 🎯 Design Principles

Adhering to these principles ensures a clean and effective homelab:

- **GitOps-first**: Changes via Git, minimizing manual commands
- **Declarative**: Define infrastructure and apps in YAML
- **Automated**: Use Flux for deployments and updates
- **Secure**: Store secrets in 1Password
- **Minimal**: Simplify deployment manifests

### 📁 Repository Structure

```
k8s/
├── orchestration/
│   ├── flux-system/          # Flux CD configuration
│   └── foundational/         # Base infrastructure
├── core/
│   ├── data/                 # Databases
│   ├── security/             # Authentication setups
│   ├── storage/              # Longhorn settings
│   └── network/              # Networking setups
└── apps/
    └── monitoring/           # Monitoring applications
```

---

*Documentation last updated 7/21/25*
