# Authentik Setup Guide

Authentik serves as the identity provider and SSO solution for the homelab, providing centralized authentication for all services.

## Overview

Authentik is deployed as a containerized application using Helm charts, integrated with external PostgreSQL and Redis databases. It provides OAuth2/OIDC authentication flows and can integrate with external identity providers.

**Key Details:**
- **Version**: 2025.6.3
- **Namespace**: `authentik`
- **URL**: https://auth.kjho.me
- **Database**: External PostgreSQL (shared cluster database)
- **Cache**: External Redis (shared cluster cache)
- **Ingress**: Cloudflare Tunnel (no direct external exposure)

## Architecture

### Core Components

**HelmRelease Configuration**
- Authentik server and worker pods
- Configuration via environment variables
- Secret management through Kubernetes secrets
- External database connections

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/helmrelease.yaml)

**Database Initialization**
- Automated database creation via Kubernetes Job
- Runs post-install/post-upgrade hooks
- Ensures `authentik` database exists in PostgreSQL cluster

📋 [View Database Init Job](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/database-init-job.yaml)

**External DNS Integration**
- Automatic DNS record management for auth.kjho.me
- Points to Cloudflare Tunnel endpoint
- Managed by External-DNS controller

📋 [View External DNS Service](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/external-dns-service.yaml)

### Dependencies

**Required Services:**
- PostgreSQL cluster (for user data, configurations)
- Redis cluster (for sessions, caching)
- 1Password Connect (for secrets management)
- Cloudflare Tunnel (for external access)
- External-DNS (for domain management)

**Secrets Required:**
- `AUTHENTIK_SECRET_KEY`: Application secret key
- `AUTHENTIK_POSTGRESQL__PASSWORD`: Database connection password
- `AUTHENTIK_REDIS__PASSWORD`: Redis connection password

📋 [View 1Password Secret Item](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/authentik-onepassword-item.yaml)

## Deployment Process

### GitOps Deployment

Authentik follows the standard GitOps deployment pattern:

1. **Namespace Creation**: Managed by foundational Kustomization
2. **Secret Sync**: 1Password Connect syncs secrets to Kubernetes
3. **Database Init**: Job ensures database exists
4. **Helm Deployment**: Flux applies HelmRelease
5. **DNS Setup**: External-DNS creates auth.kjho.me record

### Post-Deployment Configuration

After successful deployment, manual configuration is required in the Authentik web interface:

1. **Initial Setup**: Access https://auth.kjho.me and complete setup wizard
2. **Admin User**: Create initial administrative user
3. **Identity Providers**: Configure external OAuth providers (Discord, etc.)
4. **Applications**: Set up OAuth2/OIDC applications for other services
5. **Flows**: Configure authentication and authorization flows
6. **Policies**: Define access policies and user groups

## Configuration Highlights

### Database Configuration

- **Host**: `postgresql.postgresql.svc.cluster.local`
- **Database**: `authentik`
- **User**: `postgres`
- **Connection**: Internal cluster networking

### Redis Configuration

- **Host**: `redis-master.redis.svc.cluster.local`
- **Purpose**: Session storage, caching
- **Connection**: Internal cluster networking

### Security Features

- **Error Reporting**: Disabled for privacy
- **External Access**: Only via Cloudflare Tunnel
- **Secrets**: Stored in 1Password vault, synced via Connect
- **Network**: Internal service mesh communication

## Troubleshooting

### Common Issues

**Database Connection Issues**
```bash
kubectl logs -n authentik deployment/authentik-server
kubectl get pods -n postgresql
```

**Secret Sync Problems**
```bash
kubectl get onepassworditem -n authentik
kubectl describe secret authentik-secret -n authentik
```

**DNS Resolution**
```bash
kubectl logs -n external-dns deployment/external-dns
dig auth.kjho.me
```

### Useful Commands

```bash
# Check Authentik deployment status
flux get helmreleases -n authentik

# View Authentik logs
kubectl logs -n authentik -l app.kubernetes.io/name=authentik

# Check database initialization
kubectl get jobs -n authentik

# Verify external access
curl -I https://auth.kjho.me
```

## Integration Examples

### Adding New Applications

When integrating services with Authentik:

1. Create OAuth2/OIDC provider in Authentik UI
2. Configure the application to use Authentik for authentication
3. Set up appropriate flows and policies
4. Test authentication flow

### Discord OAuth Integration

Authentik can integrate with Discord for user authentication:

1. Configure Discord OAuth application
2. Add Discord provider in Authentik
3. Create authentication flow using Discord
4. Map Discord user attributes to Authentik users

---

📁 **Related Files:**
- [Authentik Kustomization](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/kustomization.yaml)
- [Authentik Namespace](https://github.com/kylejschultz/kjho.me/blob/main/k8s/orchestration/foundational/namespaces/authentik.yaml)
- [Authentik Helm Repository](https://github.com/kylejschultz/kjho.me/blob/main/k8s/orchestration/foundational/helmrepos/authentik.yaml)
