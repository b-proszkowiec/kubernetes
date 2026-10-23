# Cert-manager installation on k3s

## Prerequisites

- Install `kubectl`, Helm 3+, and `envsubst`.
- Create a local `.env` file containing following configuration:

```dotenv
DOMAIN=your-domain.net.org
ACME_EMAIL=your-email@example.com
CF_API_TOKEN=your-api-token
```

Load the environment variables before running deployment commands:

```bash
set -a
source ../.env
set +a
```

## 1. Add the Helm repository

Add the Jetstack repository if it has not already been configured:

```bash
helm repo add jetstack https://charts.jetstack.io --force-update
helm repo update
```

## 2. Create the cert-manager namespace

Create the namespace if it does not already exist:

```bash
kubectl create namespace cert-manager \
  --dry-run=client -o yaml | kubectl apply -f -
```

## 3. Create the Cloudflare API token Secret

```bash
envsubst '${DOMAIN} ${ACME_EMAIL} ${CF_API_TOKEN}' \
  < cert-manager-issuer-secret.yaml \
  | kubectl apply -f -
```

## 4. Install or upgrade cert-manager

```bash
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --values values.yaml
```

## 5. Apply the ClusterIssuer

```bash
envsubst '${CF_EMAIL} ${ACME_EMAIL}' \
  < cert-manager-cluster-issuer.yaml \
  | kubectl apply -f -
```

## 6. Configure Traefik

```bash
helm upgrade traefik traefik/traefik \
  --namespace kube-system \
  --values traefik-values.yaml
```

## 7. Verify the installation

Check the cert-manager pods:

```bash
kubectl get pods -n cert-manager
```

Check the ClusterIssuer status:

```bash
kubectl get clusterissuer cloudflare-cluster-issuer
kubectl describe clusterissuer cloudflare-cluster-issuer
```

Check the certificates:

```bash
kubectl get certificates -A
```

A correctly configured ClusterIssuer should eventually report `Ready=True`.
