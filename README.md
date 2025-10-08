# Deploying Vaultwarden on K3s/Kubernetes

This repository contains a set of Kubernetes manifests for deploying [Vaultwarden](https://github.com/dani-garcia/vaultwarden), a lightweight, unofficial Bitwarden server written in Rust. This configuration is tested on a single-node MicroK8s cluster but should be adaptable to any Kubernetes environment.

The configuration is designed to separate application configuration, metadata, and the actual media files for better data management.

<br>

## Prerequisites

1.  A working Kubernetes cluster (e.g., K3s, k0s, RKE2).
2.  The `kubectl` command-line tool configured to access your cluster.
3.  An **Ingress Controller** installed in the cluster (e.g., Traefik, NGINX Ingress).
4.  A **StorageClass** configured to dynamically provision storage volumes.
5.  An SMTP server for sending invitation emails and other notifications.

<br>

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/simon-verbois/vaultwarden-k8s
cd vaultwarden-k8s
```

### 2. Rename the files

```bash
for file in *.yaml.template; do mv "$file" "${file%.template}"; done
```

### 3. Custom the configuration files

You have to adapt the yaml file, follow the hint for each file

### 4. Deploy Vaultwarden

Apply all YAML manifests in a single command from the project root:

```bash
kubectl apply -f .
```

This will create the `vaultwarden` namespace and all the necessary resources: PVC, Secret, ConfigMap, Deployment, Service, and Ingress.

### 5. Access Vaultwarden

After a few moments, the container image will be downloaded and the pod will start. You can monitor the status with:

```bash
kubectl get pods -n vaultwarden -w
```

Once the pod is `Running`, you should be able to access your Vaultwarden instance via the URL you configured (e.g., `https://vault.your-domain.com`).

To access the admin panel, navigate to `https://vault.your-domain.com/admin` and use the plain-text password you used to generate the `ADMIN_TOKEN`.

<br>

## Maintenance

### Updating the Vaultwarden Image

The deployment uses the `vaultwarden/server:latest` image tag. To update your instance to the latest version, you can trigger a rolling update of the deployment. This will force Kubernetes to pull the newest image.

```bash
kubectl rollout restart deployment/vaultwarden-deployment -n vaultwarden
```

You can monitor the progress of the update with:

```bash
kubectl rollout status deployment/vaultwarden-deployment -n vaultwarden
```

<br>

## Uninstallation

To remove all the resources created by these manifests, run the following command:

```bash
kubectl delete -f .
```

**Note:** This will also delete the `vaultwarden` namespace. The `PersistentVolumeClaim` will be deleted, but the actual data on your storage volume might remain, depending on your `StorageClass` reclaim policy.

<br>

## License

This project is licensed under the MIT License.