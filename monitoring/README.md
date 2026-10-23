# Monitoring Installation on k3s

## Prerequisites

- Install `kubectl`, Helm 3+, and `envsubst`.
- Create a local `.env` file containing following configuration:

```dotenv
DOMAIN="you-domain.org"
GRAFANA_PWD="grafana-pwd"
```

Load the environment variables before deploying:

```bash
set -a
source ../.env
set +a
```

### 1. Add the Prometheus Community Helm repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 2. Verify the storage class (if it doesn't exist do the "nfs-storage" section first)

Make sure the `nfs-csi` StorageClass is available in the cluster:

```bash
kubectl get storageclass nfs-csi
```



## Installation

```bash
envsubst '${DOMAIN} ${GRAFANA_PWD}' < values.yaml > /tmp/monitoring-values.yaml

helm upgrade --install kpstack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values /tmp/monitoring-values.yaml
```

## Verification

Check the Helm release status:

```bash
helm status kpstack --namespace monitoring
```
