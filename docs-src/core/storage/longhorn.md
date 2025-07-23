# Longhorn Distributed Storage

Longhorn provides distributed block storage for the Kubernetes cluster, offering persistent storage with replication, snapshots, and backup capabilities.

## Overview

**Purpose:** Cloud-native distributed block storage for Kubernetes

**Key Features:**

- Distributed storage across cluster nodes
- Automatic data replication for high availability
- Volume snapshots and backups
- Web-based management interface
- CSI driver for seamless Kubernetes integration
- Cross-node data distribution

**Technical Details:**

- **Namespace**: `longhorn-system`
- **Chart**: `longhorn/longhorn`
- **Version**: v1.8.1
- **StorageClass**: `longhorn`
- **Web UI**: `https://longhorn.kjho.me`

## Architecture

### Core Components

**Longhorn Manager:**

- Runs as DaemonSet on all nodes
- Manages volume creation, deletion, and operations
- Handles replica placement and scheduling
- Provides API for volume management

**Longhorn Engine:**

- Per-volume process managing I/O operations
- Handles data replication between replicas
- Performs snapshot and backup operations
- Ensures data consistency and integrity

**Longhorn UI:**

- Web-based management interface
- Volume monitoring and management
- Node and disk management
- Backup and snapshot operations

📋 [View HelmRelease Configuration](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/storage/longhorn/helmrelease.yaml)

### Storage Architecture

**Data Distribution:**

- Volumes distributed across available nodes
- Configurable replica count (default: 3)
- Automatic replica placement based on node resources
- Support for node affinity and anti-affinity rules

**Node Requirements:**

- **nuc1**: 512GB SSD, 32GB RAM
- **nuc2**: 512GB SSD, 16GB RAM
- **nuc3**: 256GB SSD, 8GB RAM
- **nucx**: 1TB SSD, 4-8GB RAM (dynamic)

## Configuration

### Storage Classes

**Default Longhorn StorageClass:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: longhorn
provisioner: driver.longhorn.io
parameters:
  numberOfReplicas: "2"
  staleReplicaTimeout: "30"
  fromBackup: ""
  fsType: "ext4"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
```

**Key Parameters:**

- **numberOfReplicas**: Data redundancy level
- **staleReplicaTimeout**: Timeout for stale replica detection
- **fsType**: Filesystem type for volumes
- **allowVolumeExpansion**: Enable dynamic volume expansion

### Volume Management

**Persistent Volume Claims:**

Applications request storage through PVCs that automatically provision Longhorn volumes:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: app-namespace
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 10Gi
```

**Volume Features:**

- **Dynamic Provisioning**: Automatic volume creation
- **Volume Expansion**: Online volume size increase
- **Snapshots**: Point-in-time volume snapshots
- **Backups**: External backup to S3-compatible storage
- **Cloning**: Create volumes from existing volumes/snapshots

## High Availability

### Replica Management

**Replica Placement:**

- Automatic replica distribution across nodes
- Anti-affinity rules prevent replicas on same node
- Node failure tolerance based on replica count
- Automatic replica rebuilding on node recovery

**Data Consistency:**

- Synchronous replication between replicas
- Read/write quorum requirements
- Automatic corrupt replica detection
- Consistent snapshot creation across replicas

### Failure Scenarios

**Node Failure:**

- Remaining replicas continue serving I/O
- Automatic replica rebuilding on available nodes
- No data loss with sufficient replica count
- Application pods automatically reschedule

**Disk Failure:**

- Affected replicas marked as failed
- New replicas created on healthy disks
- Data rebuilt from healthy replicas
- Automatic cleanup of failed replicas

## Monitoring and Management

### Web Interface

**Dashboard Features:**

- Volume status and health monitoring
- Node and disk utilization
- Backup and snapshot management
- Performance metrics and alerts
- System event logging

**Access:**

- **URL**: https://longhorn.kjho.me
- **Authentication**: Kubernetes RBAC integration
- **Authorization**: Admin access required

### Volume Operations

**Snapshot Management:**

```bash
# Create volume snapshot
kubectl apply -f - <<EOF
apiVersion: longhorn.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: volume-snapshot
  namespace: longhorn-system
spec:
  volumeName: pvc-volume-name
EOF

# List snapshots
kubectl get volumesnapshots -n longhorn-system
```

**Backup Operations:**

- **Backup Target**: S3-compatible storage (optional)
- **Automatic Backups**: Scheduled backup policies
- **Cross-Cluster Recovery**: Restore volumes in different clusters
- **Incremental Backups**: Efficient backup storage

## Performance Optimization

### Node Configuration

**Disk Performance:**

- Use SSDs for better I/O performance
- Ensure sufficient disk space for replicas
- Monitor disk usage and health
- Consider disk locality for performance

**Network Optimization:**

- High-bandwidth network between nodes
- Low-latency connections for replication
- Monitor network utilization
- Consider dedicated storage network

### Volume Configuration

**Performance Tuning:**

- Adjust replica count based on requirements
- Use appropriate filesystem type
- Configure volume scheduling policies
- Monitor volume I/O patterns

## Troubleshooting

### Common Issues

**Volume Mount Failures:**

```bash
# Check PVC status
kubectl get pvc -A

# Check volume status
kubectl get volumes -n longhorn-system

# Check Longhorn system pods
kubectl get pods -n longhorn-system

# View volume details in UI
# Access https://longhorn.kjho.me
```

**Replica Issues:**

```bash
# Check replica status
kubectl get replicas -n longhorn-system

# View engine logs
kubectl logs -n longhorn-system -l app=longhorn-manager

# Check node disk status
kubectl get nodes -n longhorn-system -o wide
```

**Performance Problems:**

```bash
# Monitor volume I/O
kubectl top pods -A

# Check node resources
kubectl top nodes

# View Longhorn system events
kubectl get events -n longhorn-system --sort-by=.metadata.creationTimestamp
```

### Recovery Procedures

**Volume Recovery:**

1. **Identify Problem**: Check volume status in UI
2. **Assess Replicas**: Verify replica health and count
3. **Rebuild Replicas**: Trigger replica rebuilding if needed
4. **Data Verification**: Verify data integrity after recovery

**Node Recovery:**

1. **Node Restart**: Restart failed node if possible
2. **Replica Migration**: Move replicas to healthy nodes
3. **Disk Replacement**: Replace failed disks if necessary
4. **Data Rebuild**: Rebuild replicas on new storage

## Backup Strategy

### Backup Configuration

**S3 Backup Target (Optional):**

```yaml
# Configure in Longhorn settings
backupTarget: "s3://bucket-name@region/"
backupTargetCredential: "s3-credentials-secret"
```

**Backup Policies:**

- **Automated Snapshots**: Scheduled volume snapshots
- **Retention Policies**: Automatic snapshot cleanup
- **Cross-Cluster Backup**: Backup to external storage
- **Disaster Recovery**: Full cluster backup strategy

### Snapshot Management

**Snapshot Best Practices:**

- Regular snapshot schedules for critical volumes
- Retention policies to manage storage usage
- Test snapshot restoration procedures
- Document recovery procedures

## Useful Commands

```bash
# Check Longhorn system status
kubectl get pods -n longhorn-system

# List all volumes
kubectl get volumes -n longhorn-system

# Check storage classes
kubectl get storageclass

# View PVC status across namespaces
kubectl get pvc -A

# Check node storage capacity
kubectl get nodes -o custom-columns=NAME:.metadata.name,CAPACITY:.status.capacity.storage

# Monitor Longhorn manager logs
kubectl logs -n longhorn-system -l app=longhorn-manager -f

# Check volume snapshots
kubectl get volumesnapshots -n longhorn-system

# View node disk usage
kubectl get nodes -n longhorn-system -o yaml | grep -A 10 "storage"
```

## Best Practices

### Storage Management

**Volume Planning:**

- Size volumes appropriately for application needs
- Consider growth patterns and expansion requirements
- Use appropriate replica counts for data criticality
- Monitor storage usage and capacity

**Performance Optimization:**

- Place I/O intensive workloads on performant nodes
- Monitor volume performance metrics
- Use appropriate filesystem types
- Consider storage locality for performance

### Maintenance

**Regular Tasks:**

- Monitor node disk health and capacity
- Review volume usage and performance
- Test backup and recovery procedures
- Update Longhorn version regularly
- Clean up unused volumes and snapshots

**Capacity Planning:**

- Monitor cluster storage growth
- Plan for node storage expansion
- Consider data lifecycle policies
- Implement storage quotas where appropriate

---

📁 **Related Files:**

- [Longhorn HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/storage/longhorn/helmrelease.yaml)
- [Longhorn External DNS](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/storage/longhorn/external-dns-service.yaml)
- [Storage Kustomization](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/storage/kustomization.yaml)
