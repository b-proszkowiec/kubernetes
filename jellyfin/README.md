# Jellyfin Installation on k3s

## Prerequisites

- Install `kubectl`, Helm 3+, and `envsubst`.
- Create a local `.env` file containing following configuration:

```dotenv
DOMAIN=you-domain.org
```

Load the environment variables before deploying:

```bash
set -a
source ../.env
set +a
```

## 1. Create the Jellyfin namespace

Create the namespace if it does not already exist:

```bash
kubectl create namespace jellyfin \
  --dry-run=client -o yaml | kubectl apply -f -
```

## 2. Create the certificate and IngressRoute

Apply the certificate and Traefik IngressRoute manifests:

```bash
envsubst '${DOMAIN}' < jellyfin-certificate.yaml \
  | kubectl apply -f -

envsubst '${DOMAIN}' < ingress-route.yaml \
  | kubectl apply -f -
```

## 3. Create the Longhorn PersistentVolumeClaim if it doesn't exist (complete the Longhorn section first):

Create or update the PersistentVolumeClaim:

```bash
kubectl apply -f longhorn-pvc-config.yaml
```

Verify that the PVC is bound before proceeding:

```bash
kubectl get pvc -n jellyfin
```

The PVC should eventually show `STATUS=Bound`.

## 4. Install or upgrade Jellyfin

Add the Helm repository if it has not already been configured:

```bash
helm repo add utkuozdemir https://utkuozdemir.github.io/helm-charts
helm repo update
```

Install Jellyfin or upgrade the existing release:

```bash
helm upgrade --install jellyfin-nas utkuozdemir/jellyfin \
  --namespace jellyfin \
  --create-namespace \
  --values values.yaml
```

Verify the deployment:

```bash
kubectl get pods -n jellyfin
kubectl get svc -n jellyfin
kubectl rollout status deployment/jellyfin-nas -n jellyfin
```

## 5. Verify the certificate

Check the certificate status:

```bash
kubectl get certificates -n jellyfin
```

The certificate should eventually report `READY=True`.

To inspect certificate events and conditions:

```bash
kubectl describe certificates -n jellyfin
```

## Failover / Data Recovery

To restore previous Jellyfin data, copy the following files from your backup. This procedure was tested with Jellyfin version `10.10.7`.

```text
/config/config/system.xml
/config/config/users
/config/data/jellyfin.db
```

Before restoring data:

1. Stop Jellyfin to prevent database writes during recovery.
2. Restore the files into the corresponding paths in the persistent configuration volume.
3. Ensure that file ownership and permissions are appropriate for the Jellyfin container.
4. Restart Jellyfin and check the pod logs.

Example commands:

```bash
kubectl scale deployment jellyfin-nas \
  --replicas=0 -n jellyfin
```

Restore the backed-up files to the mounted configuration volume, then scale Jellyfin back up:

```bash
kubectl scale deployment jellyfin-nas \
  --replicas=1 -n jellyfin
```

**Important:** Restore only files compatible with the Jellyfin version being deployed. Back up the current configuration and database before replacing them.
