# Uptime Kuma

Uptime Kuma provides simple uptime monitoring and status pages for homelab services.

## Overview

**Purpose:** Self-hosted uptime monitoring with web interface

**Technical Details:**

- **Namespace**: `uptime-kuma`
- **Chart**: `uptime-kuma`
- **Version**: 1.5.0
- **URL**: `https://uptime.kjho.me`
- **Storage**: 10Gi Longhorn volume

## Deployment

### Configuration

**StatefulSet Deployment:**

- Single replica with persistent storage
- SQLite database for monitoring data
- Longhorn storage for data persistence

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/observability/uptime-kuma/helmrelease.yaml)

**External Access:**

- Exposed via Cloudflare Tunnel
- DNS managed by External-DNS
- No direct ingress required

📋 [View External DNS Service](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/observability/uptime-kuma/external-service.yaml)

### Monitoring Setup

**Current Monitored Services:**

- Authentik (`https://auth.kjho.me`)
- Longhorn UI (`https://longhorn.kjho.me`)
- External connectivity checks
- Database health (internal checks)

**Access & Configuration:**

- Web UI: `https://uptime.kjho.me`
- Admin interface for adding/removing monitors
- Status page generation for public visibility

## Troubleshooting

### Common Issues

```bash
# Check pod status
kubectl get pods -n uptime-kuma

# View logs
kubectl logs -n uptime-kuma statefulset/uptime-kuma

# Check storage
kubectl get pvc -n uptime-kuma

# Test external access
curl -I https://uptime.kjho.me
```

### Storage Issues

```bash
# Check Longhorn volume
kubectl get volumes -n longhorn-system | grep uptime-kuma

# Verify mount
kubectl describe pod -n uptime-kuma
```

## Useful Commands

```bash
# Monitor resource usage
kubectl top pods -n uptime-kuma

# Check external DNS
kubectl get service -n uptime-kuma

# Restart if needed
kubectl rollout restart -n uptime-kuma statefulset/uptime-kuma
```

---

📁 **Related Files:**

- [Uptime Kuma HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/observability/uptime-kuma/helmrelease.yaml)
- [External DNS Service](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/observability/uptime-kuma/external-service.yaml)
