# KJHome - Self-Hosted Kubernetes Homelab Reference

A comprehensive reference for managing and maintaining the self-hosted Kubernetes homelab. Includes setup documentation, configuration details, and recreation steps for all services.

## 📋 Documentation Index

- [**Authentik Configuration**](Authentik-Configuration) - Details on authentication flows, policies, and providers
- [**Infrastructure Overview**](Infrastructure-Overview) - Hardware, networking, and core services
- [**GitOps Workflow**](GitOps-Workflow) - Flux deployment patterns and procedures
- [**Service Directory**](Service-Directory) - Configuration for all deployed applications
- [**Troubleshooting**](Troubleshooting) - Common issues and solutions

## 🏗️ Infrastructure Overview

### Hardware
- **Cluster**: 4x Intel NUCs (lognuc1, lognuc2, lognuc3, lognucx)
- **Storage**: Longhorn distributed block storage
- **Networking**: Cloudflare Tunnels for secure ingress

### Core Technologies
- **GitOps**: Flux CD for all deployments
- **Package Management**: Helm charts for applications
- **Secrets Management**: 1Password Operator
- **Authentication**: Authentik with Discord OAuth
- **Monitoring**: Discord webhooks, UptimeKuma

### Key Services
- **PostgreSQL & Redis**: Shared databases
- **Authentik**: Identity provider
- **UptimeKuma**: Service monitoring
- **Longhorn**: Distributed storage

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

## 🔗 Quick Access URLs

- **Authentik**: https://auth.kjho.me
- **UptimeKuma**: https://uptime.kjho.me
- **Longhorn**: https://longhorn.kjho.me

---

*Documentation maintained as of July 2025*
