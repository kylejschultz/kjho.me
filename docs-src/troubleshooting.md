# Troubleshooting Guide

Common issues and solutions for the homelab infrastructure.

## Flux GitOps Issues

### Flux Not Syncing Changes

**Symptoms:**
- Changes pushed to Git but not applied to cluster
- Kustomizations stuck in "Unknown" state

**Diagnosis:**
```bash
# Check Flux system status
flux get kustomizations

# Check source controller
flux get sources git

# View source controller logs
kubectl logs -n flux-system -l app=source-controller
```

**Solutions:**

1. **Force reconciliation:**
   ```bash
   flux reconcile source git flux-system
   flux reconcile kustomization flux-system
   ```

2. **Check Git repository access:**
   ```bash
   flux get sources git -o yaml
   ```

### HelmRelease Failures

**Symptoms:**
- HelmRelease in "Failed" state
- Applications not deploying

**Diagnosis:**
```bash
# Check HelmRelease status
flux get helmreleases -A

# Get detailed error information
kubectl describe helmrelease <release-name> -n <namespace>

# Check helm controller logs
kubectl logs -n flux-system -l app=helm-controller
```

**Solutions:**

1. **Check chart repository:**
   ```bash
   flux get sources helm
   ```

2. **Suspend and resume:**
   ```bash
   flux suspend helmrelease <release-name> -n <namespace>
   flux resume helmrelease <release-name> -n <namespace>
   ```

## Database Connection Issues

### PostgreSQL Connection Problems

**Symptoms:**
- Applications unable to connect to database
- Connection timeouts or authentication failures

**Diagnosis:**
```bash
# Check PostgreSQL pod status
kubectl get pods -n postgresql

# View PostgreSQL logs
kubectl logs -n postgresql statefulset/postgresql

# Test connection from within cluster
kubectl run -it --rm debug --image=postgres:16 --restart=Never -- psql -h postgresql.postgresql.svc.cluster.local -U postgres
```

**Solutions:**

1. **Check service and endpoints:**
   ```bash
   kubectl get svc -n postgresql
   kubectl get endpoints -n postgresql
   ```

2. **Verify secrets:**
   ```bash
   kubectl get secrets -n postgresql
   kubectl describe onepassworditem -n postgresql
   ```

### Redis Connection Issues

**Symptoms:**
- Cache misses or connection errors
- Applications reporting Redis unavailability

**Diagnosis:**
```bash
# Check Redis pod status
kubectl get pods -n redis

# Test Redis connection
kubectl run -it --rm debug --image=redis:7 --restart=Never -- redis-cli -h redis-master.redis.svc.cluster.local ping
```

## Networking Problems

### Cloudflare Tunnel Issues

**Symptoms:**
- External services not accessible
- Tunnel showing as disconnected

**Diagnosis:**
```bash
# Check cloudflared pod status
kubectl get pods -n cloudflared

# View tunnel logs
kubectl logs -n cloudflared deployment/cloudflared-cloudflare-tunnel
```

**Solutions:**

1. **Restart tunnel:**
   ```bash
   kubectl rollout restart -n cloudflared deployment/cloudflared-cloudflare-tunnel
   ```

2. **Check tunnel configuration:**
   ```bash
   kubectl get configmap -n cloudflared
   ```

### DNS Resolution Problems

**Symptoms:**
- Services not resolving by name
- External DNS records not created

**Diagnosis:**
```bash
# Check external-dns logs
kubectl logs -n external-dns deployment/external-dns

# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup auth.kjho.me
```

**Solutions:**

1. **Check external-dns configuration:**
   ```bash
   kubectl get service -A | grep external-dns
   ```

2. **Verify Cloudflare credentials:**
   ```bash
   kubectl describe secret -n external-dns
   ```

## Storage Issues

### Longhorn Volume Problems

**Symptoms:**
- PVCs stuck in "Pending" state
- Application pods failing to start due to volume mount issues

**Diagnosis:**
```bash
# Check PVC status
kubectl get pvc -A

# Check Longhorn system
kubectl get pods -n longhorn-system

# Check storage classes
kubectl get storageclass
```

**Solutions:**

1. **Check Longhorn UI:**
   - Access Longhorn dashboard at configured URL
   - Review volume and node status

2. **Restart Longhorn components:**
   ```bash
   kubectl rollout restart -n longhorn-system daemonset/longhorn-manager
   ```

## Authentication Issues

### Authentik Problems

**Symptoms:**
- Unable to access Authentik UI
- Authentication flows failing
- Database connection errors

**Diagnosis:**
```bash
# Check Authentik pods
kubectl get pods -n authentik

# View Authentik logs
kubectl logs -n authentik deployment/authentik-server
kubectl logs -n authentik deployment/authentik-worker

# Check database initialization
kubectl get jobs -n authentik
```

**Solutions:**

1. **Restart Authentik components:**
   ```bash
   kubectl rollout restart -n authentik deployment/authentik-server
   kubectl rollout restart -n authentik deployment/authentik-worker
   ```

2. **Check database connectivity:**
   ```bash
   kubectl exec -it -n authentik deployment/authentik-server -- python manage.py check --database default
   ```

### 1Password Sync Issues

**Symptoms:**
- Secrets not syncing from 1Password
- OnePasswordItem resources in error state

**Diagnosis:**
```bash
# Check 1Password Connect status
kubectl get pods -n 1password-connect

# Check OnePasswordItem status
kubectl get onepassworditem -A

# View operator logs
kubectl logs -n 1password-connect deployment/onepassword-connect-operator
```

**Solutions:**

1. **Restart 1Password Connect:**
   ```bash
   kubectl rollout restart -n 1password-connect deployment/onepassword-connect
   ```

2. **Check credentials:**
   ```bash
   kubectl describe secret -n 1password-connect
   ```

## General Debugging Commands

### Cluster Health

```bash
# Node status
kubectl get nodes -o wide

# Resource usage
kubectl top nodes
kubectl top pods -A

# Events
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

### Pod Debugging

```bash
# Pod status and details
kubectl get pods -A -o wide
kubectl describe pod <pod-name> -n <namespace>

# Container logs
kubectl logs <pod-name> -n <namespace> -c <container-name>
kubectl logs <pod-name> -n <namespace> --previous

# Execute into pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
```

### Network Debugging

```bash
# Test connectivity between pods
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -qO- http://service-name.namespace.svc.cluster.local

# Check service endpoints
kubectl get endpoints -n <namespace>

# Network policies
kubectl get networkpolicies -A
```

## Emergency Procedures

### Complete Cluster Reset

!!! warning "Destructive Operation"
    Only use in development or when cluster is completely broken

```bash
# From deployment machine
ansible-playbook -i provisioning/k3s-inventory.ini provisioning/k3s-wipe.yml
ansible-playbook -i provisioning/k3s-inventory.ini provisioning/k3s-bootstrap.yml
```

### Service Rollback

```bash
# Rollback HelmRelease to previous version
flux suspend helmrelease <release-name> -n <namespace>
helm rollback <release-name> -n <namespace>
flux resume helmrelease <release-name> -n <namespace>

# Or revert Git commit
git revert <commit-hash>
git push origin main
```

---

📁 **Related Files:**

- [Discord Notifications](https://github.com/kylejschultz/kjho.me/tree/main/k8s/orchestration/flux-system/flux-notifications) - Alert configuration
- [Database Configurations](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/database) - PostgreSQL and Redis
- [Network Configurations](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/network) - Cloudflare and DNS
