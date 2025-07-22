# Redis

Redis provides high-performance in-memory data storage for caching and session management.

## Overview

**Purpose:** In-memory data structure store for caching and message brokering

**Technical Details:**

- **Namespace**: `redis`
- **Chart**: `bitnami/redis`
- **Version**: 20.3.0
- **Mode**: Master-replica replication
- **Storage**: 8Gi Longhorn volume

## Deployment

### Configuration

**StatefulSet Deployment:**

- Master-replica configuration for high availability
- Persistent storage for data durability
- ClusterIP service for internal cluster access
- Password authentication via 1Password secrets

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/redis/helmrelease.yaml)

**Connected Services:**

- Authentik (session storage)
- Application caching layer
- Internal service communication

**Internal Access:**

- Service: `redis-master.redis.svc.cluster.local:6379`
- Authentication: Password-based via Kubernetes secrets

---

📁 **Related Files:**

- [Redis HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/redis/helmrelease.yaml)

## Troubleshooting

### Common Issues

```bash
# Check pod status
kubectl get pods -n redis

# View logs for debugging
kubectl logs -n redis statefulset/redis-master

# Test connection
kubectl run -it --rm debug --image=redis:7 --restart=Never -- redis-cli -h redis-master.redis.svc.cluster.local ping

# Check storage
kubectl get pvc -n redis
```

### Performance Issues

```bash
# Monitor resource usage
kubectl top pods -n redis

# Check memory usage
kubectl exec -n redis redis-master-0 -- redis-cli info memory

# Restart if needed
kubectl rollout restart -n redis statefulset/redis-master
```

## Useful Commands

```bash
# Connect to Redis CLI
kubectl exec -it -n redis redis-master-0 -- redis-cli

# Check Redis info
kubectl exec -n redis redis-master-0 -- redis-cli info

# Monitor Redis operations
kubectl exec -n redis redis-master-0 -- redis-cli monitor
```
