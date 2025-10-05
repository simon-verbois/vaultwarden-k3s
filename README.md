# Deploying Vaultwarden on K3s/Kubernetes

This repository contains a set of Kubernetes manifests for deploying [Vaultwarden](https://github.com/dani-garcia/vaultwarden), a lightweight, unofficial Bitwarden server written in Rust, on a K3s cluster (or any other Kubernetes cluster).

The configuration is designed to be simple, clean, and easily maintainable.

## Features

  - Simple deployment via `kubectl apply`.
  - Data persistence managed by a `PersistentVolumeClaim`.
  - Centralized configuration in a `ConfigMap`.
  - Sensitive values (like the admin token) stored in a `Secret`.
  - Secure exposure via an `Ingress` (example provided for Traefik).
  - Push notifications enabled via Bitwarden's official push relay service.

## Prerequisites

1.  A working Kubernetes cluster (e.g., K3s, k0s, RKE2).
2.  The `kubectl` command-line tool configured to access your cluster.
3.  An **Ingress Controller** installed in the cluster (e.g., Traefik, NGINX Ingress).
4.  A **StorageClass** configured to dynamically provision storage volumes. K3s includes `local-path-provisioner`, which works for this configuration.
5.  An SMTP server for sending invitation emails and other notifications.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/simon-verbois/vaultwarden-k3s
cd vaultwarden-k3s
```

### 2. Copy the files

Make your own copy of the templates files with this command.

```bash
for file in *.yaml.template; do mv "$file" "${file%.template}"; done
```

### 3. Customize the configuration

You **must** adapt some files to your own environment before applying them.

  - **`02-secrets.yaml`**:

      - `ADMIN_TOKEN`: Generate a secure hash for your admin panel password. You can do this with this argon2 string generator: https://argon2.online/
        <i>Paste the generated hash here. This token is used to access the admin interface at `/admin`.</i>
      - `PUSH_INSTALLATION_ID` and `PUSH_INSTALLATION_KEY`: To enable push notifications, obtain your credentials from [https://bitwarden.com/host](https://bitwarden.com/host). Enter your email, and you will receive a unique ID and key.

    <!-- end list -->

    ```yaml
    stringData:
      ADMIN_TOKEN: "your_generated_argon2_hash_here" # <-- EDIT THIS
      PUSH_INSTALLATION_ID: "your_push_id_here" # <-- EDIT THIS
      PUSH_INSTALLATION_KEY: "your_push_key_here" # <-- EDIT THIS
    ```

<br>

  - **`03-configmap.yaml`**:

      - `DOMAIN`: Set this to the full URL where your Vaultwarden instance will be accessible.
      - `SIGNUPS_ALLOWED`: Set to `"true"` if you want to allow public registration. It is highly recommended to keep it `"false"` and invite users from the admin panel.
      - `SMTP_*`: Configure these variables to match your SMTP server settings.
      - `TZ`: Change the time zone if needed (e.g., `"America/New_York"`).
      - `PUSH_RELAY_BASE_URI`: Change the time zone if needed (e.g., `".eu or .com"`).

    <!-- end list -->

    ```yaml
    data:
      SIGNUPS_ALLOWED: "false"
      SMTP_HOST: "your.smtp.server" # <-- EDIT THIS
      SMTP_FROM: "Vaultwarden <no-reply@your-domain.com>" # <-- EDIT THIS
      SMTP_PORT: "587" # <-- EDIT THIS
      SMTP_SECURITY: "starttls" # Or "off", "force_tls" <-- EDIT THIS
      DOMAIN: "https://vault.your-domain.com" # <-- EDIT THIS
      TZ: "Europe/Paris" # <-- EDIT THIS
      PUSH_RELAY_BASE_URI: "https://push.bitwarden.eu"  # <-- EDIT THIS
      # ... other settings are likely fine as-is
    ```

<br>

  - **`04-deployment.yaml`**:

      - In this configuration, `fsGroup` is set to `1003`. Vaultwarden's official Docker image runs as a non-root user. The default UID/GID might vary. If you encounter permission issues with your persistent volume, you may need to adjust this `fsGroup` value or the permissions on the volume itself. For most `local-path` provisioners, `1003` should work.

    <!-- end list -->

    ```yaml
    # ...
    spec:
      securityContext:
        fsGroup: 1003 # Add your own user GID
    # ...
    ```

<br>

  - **`06-ingress.yaml`**:

      - Modify the `host` to use your own domain name. This must match the `DOMAIN` variable in the ConfigMap.
      - Adapt the ingress `annotations` for your Ingress Controller. The example uses Traefik and assumes your TLS certificate resolver is named `ovhresolver`. Change this to match your setup.

    <!-- end list -->

    ```yaml
    metadata:
      # ...
      annotations:
        traefik.ingress.kubernetes.io/router.tls.certresolver: your-certresolver-name # <-- EDIT THIS
    spec:
      rules:
      - host: "vault.your-domain.com" # <-- EDIT THIS
        # ...
      tls:
      - hosts:
        - "vault.your-domain.com" # <-- EDIT THIS
    ```

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

## Uninstallation

To remove all the resources created by these manifests, run the following command:

```bash
kubectl delete -f .
```

**Note:** This will also delete the `vaultwarden` namespace. The `PersistentVolumeClaim` will be deleted, but the actual data on your storage volume might remain, depending on your `StorageClass` reclaim policy.

## License

This project is licensed under the MIT License.