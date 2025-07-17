<p align="center"><img src="./logo_transparent.png" alt="Home Kubernetes Logo" width="300"/></p>

# KJHo.me: *A Declarative Kubernetes Homelab*

This repository serves as the single source of truth for my self-hosted Kubernetes homelab, built on GitOps principles with Flux CD. Everything is managed declaratively through code, with zero manual kubectl commands in production.

## Architecture Overview

### Core Infrastructure Stack
- **🔄 FluxCD**: GitOps continuous delivery managing all deployments
- **🔐 1Password Connect**: Direct secret synchronization from 1Password vault using Connect Operator
- **🌐 Cloudflare Tunnel**: Secure ingress without exposed ports via Cloudflare's edge network
- **🌍 External-DNS**: Automated DNS record management with Cloudflare integration
- **📦 Helm**: Package management for all applications
- **💾 Longhorn**: Distributed block storage with replication and snapshots

### Application Services
- **🗃️ PostgreSQL**: Primary database for applications
- **⚡ Redis**: Cache and session store for applications
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

## Deployment

### Prerequisites
- Physical nodes running Ubuntu with SSH access
- 1Password vault with required secrets configured
- GitHub repository access with personal access token
- Ansible installed on deployment machine

### Required Environment Variables
Before deploying, set the following environment variables:

```bash
# 1Password Connect API Token
export OP_CONNECT_TOKEN="your-1password-connect-api-token"

# 1Password Connect Credentials File
export OP_CONNECT_CREDENTIALS_FILE=$(cat /path/to/1password-credentials.json)

# GitHub Personal Access Token (for Flux bootstrap)
export GITHUB_PAT="your-github-personal-access-token"
```

### Deployment Steps

1. **Clone the repository**:
   ```bash
   git clone https://github.com/kylejschultz/kjho.me.git
   cd kjho.me
   ```

2. **Configure environment variables** (see above)

3. **Update inventory** if needed:
   ```bash
   # Edit provisioning/k3s-inventory.ini with your node details
   ```

4. **Deploy the cluster**:
   ```bash
   ansible-playbook -i provisioning/k3s-inventory.ini provisioning/k3s-bootstrap.yml
   ```

### What the Deployment Does
- **Installs K3s** on all nodes with proper networking
- **Deploys 1Password Connect Operator** for secret management
- **Bootstraps Flux** for GitOps continuous delivery
- **Applies all manifests** from this repository automatically
- **Sets up Discord notifications** for cluster monitoring

### Post-Deployment
After deployment completes:
- All applications will be automatically deployed via GitOps
- Secrets will be synced from 1Password vault
- Cloudflare Tunnel will be established for secure ingress
- DNS records will be automatically managed via External-DNS
- Services will be accessible via configured domain endpoints

## Adding New Domain Endpoints

To expose a new service through the Cloudflare Tunnel with automatic DNS management:

### Step 1: Configure the Application
Ensure the application has a ClusterIP service (this is typically created automatically by Helm charts):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: my-namespace
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

### Step 2: Add to Cloudflare Tunnel Configuration
Update the Cloudflare Tunnel ingress configuration in `k8s/core/network/cloudflared/helmrelease.yaml`:

```yaml
cloudflare:
  ingress:
    - hostname: existing-service.domain.tld
      service: http://existing-service.existing-namespace.svc.cluster.local:80
    - hostname: my-app.domain.tld  # Add your new domain
      service: http://my-app.my-namespace.svc.cluster.local:80
```

### Step 3: Create ExternalName Service for DNS Management
Create an ExternalName service to tell External-DNS to manage the domain:

```yaml
# k8s/core/[category]/[app]/external-dns-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-external-dns
  namespace: my-namespace
  annotations:
    # Tell External-DNS to create DNS records for this service
    external-dns.alpha.kubernetes.io/hostname: "my-app.domain.tld"
    # Point the DNS record to your tunnel's CNAME target
    external-dns.alpha.kubernetes.io/target: "TUNNEL_ID.cfargotunnel.com"
spec:
  type: ExternalName
  externalName: "TUNNEL_ID.cfargotunnel.com"
```

### Step 4: Add to Kustomization
Include the ExternalName service in your kustomization.yaml:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - helmrelease.yaml
  - external-dns-service.yaml  # Add this line
```

### Step 5: Deploy Changes
Commit and push your changes. Flux will automatically:
1. Update the Cloudflare Tunnel configuration
2. Create DNS records via External-DNS
3. Enable proxying through Cloudflare

### Verification
After deployment:
- Check that DNS records are created: `dig my-app.domain.tld`
- Verify External-DNS logs: `kubectl logs -n external-dns -l app.kubernetes.io/name=external-dns`
- Test accessibility: Visit `https://my-app.domain.tld` in your browser

## Cluster Management

### Complete Cluster Wipe
To completely remove K3s from all nodes and start fresh:

```bash
ansible-playbook -i provisioning/k3s-inventory.ini provisioning/k3s-wipe.yml
```

### What the Wipe Script Does
- **Runs uninstall scripts**: Executes K3s server and agent uninstall scripts
- **Removes directories**: Cleans up `/etc/rancher`, `/var/lib/rancher`, `/var/lib/kubelet`, and other K3s directories
- **Cleans network interfaces**: Removes CNI, Flannel, and other cluster network interfaces
- **Purges iptables rules**: Removes K3s-related firewall rules
- **Removes binaries**: Deletes K3s binaries and CLI tools
- **Cleans local config**: Removes local kubeconfig file

⚠️ **Warning**: This operation is destructive and will completely remove the K3s cluster. All data, applications, and configurations will be lost. Use with caution.
