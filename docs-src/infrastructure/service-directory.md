# Service Directory

## Summary

This directory outlines the foundational services within the homelab, each playing a crucial role in the overall architecture. The configuration aligns with GitOps and declarative management practices, ensuring reliability and scalability.

Explore each linked configuration for specific deployment details. The homelab's infrastructure is set up for high availability, security, and ease of management.


## Core Services

### PostgreSQL

**Role:** Primary database for various applications.

- **Chart**: Bitnami/PostgreSQL
- **Namespace**: `postgresql`
- **Storage**: Provisioned via Longhorn

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/postgresql/helmrelease.yaml)

### Redis

**Role:** Caching and message brokering.

- **Chart**: Bitnami/Redis
- **Namespace**: `redis`
- **Storage**: No persistent storage required

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/database/redis/helmrelease.yaml)

## Networking

### Cloudflare Tunnel

**Role:** Secure ingress without open ports.

- **Chart**: cloudflare/cloudflared
- **Namespace**: `cloudflared`

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/cloudflared/helmrelease.yaml)

### External DNS

**Role:** Automated DNS record management.

- **Chart**: Bitnami/External-DNS
- **Namespace**: `external-dns`

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/network/external-dns/helmrelease.yaml)

## Observability

### Uptime Kuma

**Role:** Monitoring and status pages.

- **Chart**: Uptime-Kuma
- **Namespace**: `uptime-kuma`

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/observability/uptime-kuma/helmrelease.yaml)

## Security

### Authentik

**Role:** Identity provider and SSO.

- **Chart**: Authentik
- **Namespace**: `authentik`

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/security/authentik/helmrelease.yaml)

## Storage

### Longhorn

**Role:** Distributed block storage.

- **Chart**: Longhorn
- **Namespace**: `longhorn-system`

📋 [View HelmRelease](https://github.com/kylejschultz/kjho.me/blob/main/k8s/core/storage/longhorn/helmrelease.yaml)

---

📁 **Related Files:**
- [Database HelmReleases](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/database)
- [Network HelmReleases](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/network)
- [Observability HelmReleases](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/observability)
- [Security HelmReleases](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/security)
- [Storage HelmReleases](https://github.com/kylejschultz/kjho.me/tree/main/k8s/core/storage)
