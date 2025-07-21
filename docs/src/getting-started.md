# Getting Started
## 🚀 Recreation Steps

For recreating or understanding this setup:

1. Review the [Infrastructure Overview](Infrastructure-Overview) for architecture insights.
2. Understand [GitOps Workflow](GitOps-Workflow) for deployment methods.
3. Examine [Authentik Configuration](Authentik-Configuration) for authentication setup.
4. Refer to the [Service Directory](Service-Directory) for application configurations.

## 📁 Repository Structure

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

## 🎯 Design Principles

Adhering to these principles ensures a clean and effective homelab:

- **GitOps-first**: Changes via Git, minimizing manual commands
- **Declarative**: Define infrastructure and apps in YAML
- **Automated**: Use Flux for deployments and updates
- **Secure**: Store secrets in 1Password
- **Minimal**: Simplify deployment manifests
