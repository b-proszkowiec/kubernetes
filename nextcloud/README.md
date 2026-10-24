# Nextcloud Installation on k3s

## Prerequisites

- Install `kubectl`, Helm 3+, and `envsubst`.
- Ensure that the NFS provisioner is installed and operational. If it is not installed, follow the instructions in the `nfs-storage` section.

  Verify that the NFS components are running:

  ```bash
  kubectl get pods -A | grep nfs
  ```

- Create a local `.env` file containing following configuration:

  ```dotenv
  DOMAIN=your-domain.org
  NEXTCLOUD_PWD="admin-password"
  NEXTCLOUD_MDB_PWD="maria-db-pwd"
  NEXTCLOUD_MDBR_PWD="maria-db-root-pwd"
  NEXTCLOUD_MDBREP_PWD="maria-db-rep-pwd"
  ```

Load the environment variables before deploying:

```bash
set -a
source ../.env
set +a
```

## 1. Create the Nextcloud namespace

```bash
kubectl create namespace nextcloud \
  --dry-run=client -o yaml | kubectl apply -f -
```

## 2. Create the PersistentVolumeClaims if doesn't exist (complete the nfs-storage section first)

Apply the PVC manifests for the Nextcloud application and database:

```bash
kubectl apply -f nfs-pvc-app.yaml -f nfs-pvc-db.yaml
```

Verify that the PVCs have been created:

```bash
kubectl get pvc -n nextcloud
```

Wait until the PVCs show `STATUS=Bound` before proceeding.

## 3. Create the database Secret

```bash
envsubst '${NEXTCLOUD_MDB_PWD} ${NEXTCLOUD_MDBR_PWD} ${NEXTCLOUD_MDBREP_PWD}' \
  < db-secret.yaml \
  | kubectl apply -f -
```

## 4. Install or upgrade Nextcloud

Ensure that the Nextcloud Helm repository is configured and up to date:

```bash
helm repo update
```

Install Nextcloud or upgrade the existing release:

```bash
envsubst '${DOMAIN} ${NEXTCLOUD_PWD} ${NEXTCLOUD_MDB_PWD}' < values.yaml > /tmp/nextcloud-values.yaml

helm upgrade --install nextcloud-nas nextcloud/nextcloud \
  --namespace nextcloud \
  --values /tmp/nextcloud-values.yaml
```

Check the release and Kubernetes resources:

```bash
helm status nextcloud-nas -n nextcloud
kubectl get pods,svc,pvc -n nextcloud
```

## 5. Configure the certificate and IngressRoute

Apply the certificate and Traefik IngressRoute manifests:

```bash
envsubst '${DOMAIN}' < nextcloud-certificate.yaml | kubectl apply -f -
envsubst '${DOMAIN}' < ingress-route.yaml | kubectl apply -f -
```

Verify the certificate status:

```bash
kubectl get certificates -n nextcloud
kubectl describe certificates -n nextcloud
```

# Management and Maintenance.

## Uninstall Nextcloud

To uninstall the Helm release:

```bash
helm uninstall nextcloud-nas -n nextcloud
```

## Troubleshooting

## 1. If the installation gets stuck while creating the `nextcloud-nas-redis-replicas-0` pod, inspect the pod and its events first:

```bash
kubectl describe pod nextcloud-nas-redis-replicas-0 -n nextcloud
kubectl get events -n nextcloud --sort-by=.lastTimestamp
kubectl get pods -n nextcloud -o wide
```

Check the logs if the container has started:

```bash
kubectl logs nextcloud-nas-redis-replicas-0 -n nextcloud
```

## 2. Nextcloud deployment pod stuck in CrashLoopBackOff during an image upgrade

During the image upgrade, you may encounter the following Kubernetes event:

```
  Warning  FailedPostStartHook  23s (x3 over 103s)  kubelet            spec.containers{nextcloud}: PostStartHook failed
```

Solution: If you have configured a postStartCommand in values.yaml, comment it out temporarily and redeploy Nextcloud. This allows the container to start without the post-start hook interfering with the startup or migration process.
