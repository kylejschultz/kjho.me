<div align="center">
  <img src="https://github.com/kylejschultz/kjho.me/raw/docs/assets/images/logo_transparent.png" alt="KJHome Logo" width="200" />
</div>

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

---

*Documentation last updated 7/21/25*
