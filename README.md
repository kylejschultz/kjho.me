<p align="center"><img src="./logo_transparent.png" alt="Home Kubernetes Logo" width="300"/></p>

# KJHo.me: *A Declarative Kubernetes Homelab*

This repo acts as the source of truth for my HomeLab, running on a set of NUCs along with a worker node VM.

## Current Deployment
- Flux for GitOps automation
- External-Secrets with 1Password Provider for Secret Management
- Traefik for Ingress

## Hardware
The lab consists of:

- **Intel NUCs (x3)**:
    - **nuc1**: 2 Cores, 32GB RAM, 512GB SSD
    - **nuc2**: 2 Cores, 16GB RAM, 512GB SSD
    - **nuc3**: 2 Cores, 8GB RAM, 256GB SSD
- **Worker Node VM**:
    - **nucx**: 4 core, 4-8GB RAM, 1TB SSD
        - Virtualized Ubuntu install on my gaming rig with dynamic RAM allocation
        - Intended for heavy-cpu workloads that would overpower the Celeron processors in the NUCs.
        - Will also act as storage controller, with up to 4 1TB SSDs for backing storage.