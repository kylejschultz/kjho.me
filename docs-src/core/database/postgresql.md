# PostgreSQL

PostgreSQL provides relational database services for applications requiring structured data storage.

## Overview

**Purpose:** Open-source relational database management system

**Technical Details:**

- **Namespace**: `postgresql`
- **Chart**: `bitnami/postgresql`
- **Version**: 15.5.17
- **Storage**: Longhorn persistent volumes
- **Access**: Internal cluster only

## Deployment

### Configuration

**StatefulSet Deployment:**

- Single primary instance with persistent storage
- Automatic backup and recovery capabilities
- Password authentication via 1Password secrets
- Internal ClusterIP service for cluster access

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/postgresql/helmrelease.yaml)

**Connected Services:**

- Authentik (user and session data)
- Future applications requiring relational storage

**Internal Access:**

- Service: `postgresql.postgresql.svc.cluster.local:5432`
- Database: Multiple databases per application
- Authentication: Username/password via Kubernetes secrets

## Troubleshooting

### Common Issues

```bash
# Check pod status
kubectl get pods -n postgresql

# View pod logs
kubectl logs -n postgresql statefulset/postgresql

# Test database connection
kubectl run -it --rm debug --image=postgres:16 --restart=Never -- psql -h postgresql.postgresql.svc.cluster.local -U postgres

# Check storage
kubectl get pvc -n postgresql
```

### Performance Issues

```bash
# Monitor resource usage
kubectl top pods -n postgresql

# Check database connections
kubectl exec -n postgresql statefulset/postgresql -- psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"

# Restart if needed
kubectl rollout restart -n postgresql statefulset/postgresql
```

## Useful Commands

```bash
# Connect to PostgreSQL
kubectl exec -it -n postgresql statefulset/postgresql -- psql -U postgres

# Create database backup
kubectl exec -n postgresql statefulset/postgresql -- pg_dump -U postgres database_name > backup.sql

# Monitor active connections
kubectl exec -n postgresql statefulset/postgresql -- psql -U postgres -c "SELECT * FROM pg_stat_activity;"
```

---

📁 **Related Files:**

- [PostgreSQL HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/postgresql/helmrelease.yaml)
- [Database Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/postgresql/postgresql-onepassword-item.yaml)
