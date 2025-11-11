# Vaultwarden Kubernetes Deployment

This repository provides Kubernetes manifests to deploy [Vaultwarden](https://github.com/dani-garcia/vaultwarden), a lightweight Bitwarden-compatible server. Tested on MicroK8s, but adaptable to any Kubernetes cluster.

## Prerequisites

- Kubernetes cluster (e.g., K3s, k0s, RKE2)
- `kubectl` configured for your cluster
- Ingress controller (e.g., Traefik, NGINX)
- StorageClass for persistent volumes
- SMTP server for notifications

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/simon-verbois/vaultwarden-k8s
   cd vaultwarden-k8s
   ```

2. Rename template files:
   ```bash
   for file in *.yaml.template; do mv "$file" "${file%.template}"; done
   ```

3. Customize the YAML files according to the comments in each file.

4. Deploy:
   ```bash
   kubectl apply -f .
   ```
   This creates the `vaultwarden` namespace and all resources (PVC, Secret, ConfigMap, Deployment, Service, Ingress).

5. Access Vaultwarden:
   - Monitor pods: `kubectl get pods -n vaultwarden -w`
   - Once running, access at your configured URL (e.g., `https://vault.example.org`)
   - Admin panel: `https://vault.example.org/admin` (use the password for `ADMIN_TOKEN`)

## Maintenance

### Update Vaultwarden

The deployment uses `vaultwarden/server:latest`. To update:

```bash
kubectl rollout restart deployment/vaultwarden-deployment -n vaultwarden
kubectl rollout status deployment/vaultwarden-deployment -n vaultwarden
```

## Uninstallation

Remove all resources:
```bash
kubectl delete -f .
```

**Note:** The namespace and PVC will be deleted. Data may persist based on your StorageClass reclaim policy.

## License

MIT License
