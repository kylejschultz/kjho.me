# Cloudflare Tunnel

Cloudflare Tunnel provides secure ingress to the homelab without requiring open ports on the home network or exposing services directly to the internet.

## Overview

**Purpose:** Secure, encrypted tunnel from Cloudflare's edge to internal services

**Key Benefits:**

- No open ports required on home router
- Automatic SSL/TLS termination
- DDoS protection via Cloudflare's network
- Traffic encryption between edge and origin
- Zero-trust network access

**Technical Details:**

- **Namespace**: `cloudflared`
- **Chart**: `cloudflare/cloudflare-tunnel`
- **Version**: 0.3.2
- **URL**: Tunnel endpoints configured per service

## Architecture

### Tunnel Components

**Cloudflared Daemon:**

- Runs as Kubernetes deployment (2 replicas for HA)
- Establishes secure connection to Cloudflare edge
- Routes traffic based on hostname/path rules
- Handles SSL termination at Cloudflare edge

**Ingress Configuration:**

- Hostname-based routing to internal services
- Service discovery via Kubernetes DNS
- Load balancing across multiple tunnel replicas
- Health checks and automatic failover

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/cloudflared/helmrelease.yaml)

### Traffic Flow

1. **External Request**: User accesses `https://service.kjho.me`
2. **Cloudflare Edge**: Request hits Cloudflare's global network
3. **Tunnel Authentication**: Edge validates tunnel connection
4. **Traffic Routing**: Request routed through encrypted tunnel
5. **Service Resolution**: Cloudflared resolves internal Kubernetes service
6. **Response**: Response flows back through tunnel to user

## Configuration

### Tunnel Setup

**Prerequisites:**

- Cloudflare account with domain management
- Tunnel token from Cloudflare dashboard
- 1Password secret containing tunnel credentials

**Tunnel Token Storage:**

```yaml
# Stored in 1Password and synced via OnePasswordItem
apiVersion: onepassword.com/v1
kind: OnePasswordItem
metadata:
  name: cloudflared-token
  namespace: cloudflared
spec:
  itemPath: "vaults/Homelab/items/Cloudflare Tunnel Token"
```

📋 [View 1Password Secret Item](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/cloudflared/cloudflared-token-onepassword-item.yaml)

### Service Routing

**Current Service Mappings:**

- **Authentik**: `auth.kjho.me` → `authentik-server.authentik.svc.cluster.local:80`
- **UptimeKuma**: `uptime.kjho.me` → `uptime-kuma.uptime-kuma.svc.cluster.local:3001`
- **Longhorn**: `longhorn.kjho.me` → `longhorn-frontend.longhorn-system.svc.cluster.local:80`

**Adding New Services:**

1. **Configure Ingress Rules**: Add hostname mapping in HelmRelease
2. **Create ExternalName Service**: For DNS management via External-DNS
3. **Test Connectivity**: Verify internal service accessibility
4. **DNS Propagation**: Wait for DNS records to propagate

### High Availability

**Deployment Strategy:**

- **Replicas**: 2 pods for redundancy
- **Anti-Affinity**: Spread across different nodes
- **Health Checks**: Kubernetes liveness/readiness probes
- **Restart Policy**: Always restart on failure

**Monitoring:**

- **Pod Status**: Monitor via Kubernetes metrics
- **Tunnel Status**: Cloudflare dashboard shows tunnel health
- **Traffic Metrics**: Available in Cloudflare Analytics

## External DNS Integration

### Automatic DNS Management

Services exposed via Cloudflare Tunnel use External-DNS for automated record management:

**ExternalName Service Pattern:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-external-dns
  namespace: service-namespace
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "service.kjho.me"
    external-dns.alpha.kubernetes.io/target: "TUNNEL_ID.cfargotunnel.com"
spec:
  type: ExternalName
  externalName: "TUNNEL_ID.cfargotunnel.com"
```

**Benefits:**

- Automatic DNS record creation/deletion
- Consistent tunnel endpoint management
- GitOps-managed DNS configuration
- No manual DNS record management

## Security Features

### Access Control

**Cloudflare Access (Optional):**

- Can integrate with Cloudflare Access for additional authentication
- Supports OIDC integration with Authentik
- Granular access policies per application

**Network Security:**

- All traffic encrypted in transit
- No direct exposure of internal services
- Rate limiting and DDoS protection
- Geographic access restrictions available

### Certificate Management

**SSL/TLS:**

- Automatic SSL certificate provisioning
- Cloudflare Universal SSL
- HSTS and security headers
- Perfect Forward Secrecy

## Troubleshooting

### Common Issues

**Tunnel Disconnected:**

```bash
# Check pod status
kubectl get pods -n cloudflared

# View tunnel logs
kubectl logs -n cloudflared -l app.kubernetes.io/name=cloudflare-tunnel

# Restart tunnel
kubectl rollout restart -n cloudflared deployment/cloudflared-cloudflare-tunnel
```

**Service Not Accessible:**

```bash
# Test internal connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -qO- http://service.namespace.svc.cluster.local

# Check service endpoints
kubectl get endpoints -n namespace

# Verify tunnel configuration
kubectl get configmap -n cloudflared
```

**DNS Issues:**

```bash
# Check external-dns logs
kubectl logs -n external-dns -l app.kubernetes.io/name=external-dns

# Verify DNS records
dig service.kjho.me

# Check ExternalName service
kubectl get service -n namespace
```

### Performance Optimization

**Bandwidth Management:**

- Monitor traffic via Cloudflare dashboard
- Configure appropriate timeout values
- Use connection pooling for high-traffic services

**Latency Optimization:**

- Ensure tunnel pods run on performant nodes
- Monitor connection quality metrics
- Consider multiple tunnel instances for load distribution

## Useful Commands

```bash
# Check tunnel status
kubectl get pods -n cloudflared -o wide

# View tunnel configuration
kubectl describe configmap -n cloudflared

# Monitor tunnel logs
kubectl logs -n cloudflared -f -l app.kubernetes.io/name=cloudflare-tunnel

# Test service connectivity
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- curl -I https://service.kjho.me

# Check all external DNS services
kubectl get service -A | grep ExternalName

# Force DNS record update
kubectl annotate service service-external-dns -n namespace external-dns.alpha.kubernetes.io/force-update="$(date)"
```

## Best Practices

### Configuration Management

**Ingress Rules:**

- Use consistent hostname patterns
- Document all service mappings
- Test internal connectivity before exposing
- Monitor tunnel capacity and performance

**Security:**

- Regularly rotate tunnel tokens
- Monitor access logs via Cloudflare dashboard
- Use appropriate timeout and rate limiting
- Consider additional authentication layers

### Maintenance

**Regular Tasks:**

- Monitor tunnel health and connectivity
- Review Cloudflare dashboard for issues
- Update tunnel configuration as services change
- Test failover scenarios periodically

---

📁 **Related Files:**

- [Cloudflared HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/cloudflared/helmrelease.yaml)
- [Tunnel Token Secret](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/cloudflared/cloudflared-token-onepassword-item.yaml)
- [External-DNS Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/external-dns/helmrelease.yaml)
- [Service External DNS Examples](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core)
