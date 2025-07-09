<p align="center"><img src="./logo_transparent.png" alt="Home Kubernetes Logo" width="300"/></p>

# KJHo.me: *A Declarative Kubernetes Homelab*

This repository serves as the single source of truth for my self-hosted Kubernetes homelab, built on GitOps principles with Flux CD. Everything is managed declaratively through code, with zero manual kubectl commands in production.

## Architecture Overview

### Core Infrastructure Stack
- **🔄 FluxCD**: GitOps continuous delivery managing all deployments
- **🔐 External-Secrets**: Automated secret synchronization from 1Password vault
- **🌐 Traefik**: Ingress controller with automatic service discovery
- **🔒 cert-manager**: Automated SSL certificate management via Cloudflare
- **📦 Helm**: Package management for all applications
- **💾 Longhorn**: Distributed block storage with replication and snapshots

### Application Services
- **🔑 Authentik**: Identity provider and single sign-on (SSO)
- **🗃️ PostgreSQL**: Primary database for Authentik and other applications
- **⚡ Redis**: Cache and session store for Authentik
- **📡 Cloudflare DDNS**: Dynamic DNS updates for external access
- **📊 Uptime Kuma**: Self-hosted monitoring and status pages
- **🔔 Discord Notifications**: Automated cluster alerts and status updates

### GitOps Workflow
- **Declarative**: All infrastructure and applications defined in YAML manifests
- **Automated**: Flux handles deployment, upgrades, and reconciliation
- **Secure**: Secrets stored in 1Password, never in Git
- **Validated**: Pre-commit hooks ensure manifest quality
- **Incremental**: Small, frequent commits over large changes

### Key Design Principles
- **GitOps-First**: All changes flow through Git, avoiding manual interventions
- **Minimal Complexity**: Keep deployment manifests simple and maintainable
- **Latest Versions**: Use current Helm chart and application versions
- **Separation of Concerns**: Individual files for each Kubernetes component

## Hardware Infrastructure

### Physical Nodes
- **🖥️ Intel NUCs (x3)**: Core cluster nodes with Celeron processors
  - **lognuc1**: 2 cores, 32GB RAM, 512GB SSD
  - **lognuc2**: 2 cores, 16GB RAM, 512GB SSD
  - **lognuc3**: 2 cores, 8GB RAM, 256GB SSD
- **💻 Worker VM**: High-performance node for compute-intensive workloads
  - **lognucx**: 4 cores, 4-8GB RAM, 1TB SSD (dynamic allocation)
