# External-DNS

External-DNS automatically manages DNS records in Cloudflare for Kubernetes services and ingresses.

## Overview

**Purpose:** Automated DNS record management for Kubernetes resources

**Technical Details:**

- **Namespace**: `external-dns`
- **Chart**: `external-dns`
- **Version**: 1.15.0
- **Provider**: Cloudflare
- **Domain**: `kjho.me`

## Deployment

### Configuration

**DNS Management:**

- Automatically creates/updates DNS records for services with annotations
- Uses Cloudflare API for record management
- Proxied records (orange cloud) enabled by default
- TXT record registry for ownership tracking

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/external-dns/helmrelease.yaml)

**Authentication:**

- Cloudflare API token stored in 1Password
- Secret managed by External Secrets Operator
- Scoped to `kjho.me` domain only

📋 [View 1Password Secret](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/external-dns/external-dns-onepassword-item.yaml)

### Managed Services

**Current DNS Records:**

- `auth.kjho.me` → Authentik
- `longhorn.kjho.me` → Longhorn UI
- `uptime.kjho.me` → Uptime Kuma
- Cloudflare Tunnel CNAME records

**Service Annotation Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    external-dns.alpha.kubernetes.io/hostname: service.kjho.me
```

## Troubleshooting

### Common Issues

```bash
# Check external-dns pod status
kubectl get pods -n external-dns

# View logs for DNS operations
kubectl logs -n external-dns deployment/external-dns

# Check Cloudflare API connectivity
kubectl logs -n external-dns deployment/external-dns | grep -i cloudflare

# Verify secret exists
kubectl get secret -n external-dns external-dns-cloudflare-secret
```

### DNS Record Issues

```bash
# Check TXT records for ownership
dig TXT kjho.me | grep kjhome

# Verify service annotations
kubectl get service -A -o yaml | grep external-dns

# Check domain filtering
kubectl logs -n external-dns deployment/external-dns | grep "kjho.me"
```

### Cloudflare API Issues

```bash
# Test API token (do not echo token value)
kubectl describe secret -n external-dns external-dns-cloudflare-secret

# Check external secrets sync
kubectl get externalsecret -n external-dns
```

## Useful Commands

```bash
# Monitor DNS sync operations
kubectl logs -n external-dns deployment/external-dns -f

# Check resource usage
kubectl top pods -n external-dns

# Restart external-dns
kubectl rollout restart -n external-dns deployment/external-dns

# Force immediate sync
kubectl annotate service <service-name> external-dns.alpha.kubernetes.io/force-update=$(date +%s)
```

---

📁 **Related Files:**

- [External-DNS HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/external-dns/helmrelease.yaml)
- [1Password Secret Config](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/external-dns/external-dns-onepassword-item.yaml)
