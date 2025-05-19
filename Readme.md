# Plane Deployment with ArgoCD

This repository contains the configuration for deploying [Plane](https://plane.so/) using ArgoCD in a Kubernetes cluster.

## Prerequisites

- Kubernetes cluster with ArgoCD installed
- `kubectl` configured to access your cluster
- Helm installed (for managing the Plane chart)
- Cert-manager installed (if using TLS/SSL)

## Project Structure

The configuration consists of two main parts:

1. **AppProject**: Defines the project scope and permissions
2. **Application**: Defines the actual Plane deployment

## Deployment Configuration

### AppProject (`plane-project`)

- **Source Repositories**:
  - `https://github.com/vladoz77/plane-cd.git` (custom configurations)
  - `https://helm.plane.so/` (official Plane Helm chart)
  
- **Destinations**:
  - Deploys to `plane` namespace in the same cluster
  
- **Permissions**:
  - Allows all resources in the namespace (`namespaceResourceWhitelist`)
  - Allows all cluster resources (`clusterResourceWhitelist`)
  - Warns about orphaned resources

### Application (`plane`)

- **Sources**:
  - Custom values from the GitHub repository
  - Plane Helm chart (version 1.1.1)
  
- **Sync Policy**:
  - Automated sync enabled
  - Creates namespace if not exists
  - Prunes resources last during sync
  
- **Ingress Configuration**:
  - Host: `plane.home-local.site`
  - TLS with cert-manager using `letsencrypt-issuer`

## Deployment Steps

1. Apply the AppProject:
   ```bash
   kubectl apply -f plane-project.yaml -n argocd
   ```

2. Apply the Application:
   ```bash
   kubectl apply -f plane-application.yaml -n argocd
   ```

3. Monitor the deployment in ArgoCD UI or using:
   ```bash
   argocd app get plane
   ```

## Customization

To customize the deployment:

1. Edit the `values.yaml` in the GitHub repository
2. Update the `targetRevision` in the Application to point to your desired chart version
3. Modify the ingress host and annotations as needed

## Accessing Plane

After successful deployment, Plane will be accessible at:
- `https://plane.home-local.site` (if DNS is properly configured)

## Troubleshooting

- Check ArgoCD sync status:
  ```bash
  argocd app sync plane
  argocd app get plane
  ```

- Check Kubernetes resources:
  ```bash
  kubectl get all -n plane
  ```

- View logs for specific components:
  ```bash
  kubectl logs -n plane <pod-name>
  ```

## Maintenance

To update Plane:

1. Update the `targetRevision` in the Application to the desired version
2. ArgoCD will automatically sync the changes (if automated sync is enabled)

For manual sync:
```bash
argocd app sync plane
```