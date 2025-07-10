# Archived Deployments

This directory contains Kubernetes manifests and configurations for deployments that are no longer active in the cluster but are preserved for future reference or potential redeployment.

## Contents

### authentik/
Complete Authentik identity provider configuration including:
- HelmRelease with ingress and configuration
- External secrets for authentication, Redis, and PostgreSQL
- Namespace and Helm repository definitions
- Archived on: 2025-01-10
- Reason: Too complex for current needs, focusing on simpler setup first

## Usage

Files in this directory are **not** watched by Flux and will **not** be deployed to the cluster. To reactivate any archived deployment:

1. Review and update the manifests as needed
2. Move the files back to their appropriate locations in the active k8s/ directory structure
3. Update the relevant kustomization.yaml files to include the resources
4. Commit and push to trigger Flux reconciliation

## Important Notes

- These configurations may be outdated and should be reviewed before redeployment
- Secret references may need to be updated if the 1Password vault structure has changed
- Helm chart versions should be updated to current releases
- Dependencies and prerequisites should be verified before reactivation
