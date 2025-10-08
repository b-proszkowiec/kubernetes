# Cert-manager instalation on k3s

## Preconditions

- Add helm repository if not exist:

```
helm repo add jetstack https://charts.jetstack.io --force-update
```

- Create namespace cert-manager:

```
kubectl create namespace cert-manager
```

- Create token as secret:

```
kubectl apply -f cert-manager-issuer-secret.yaml
```

- Install cert-manager application:

```
helm install cert-manager --namespace cert-manager jetstack/cert-manager --values values.yaml
```

- Install custom Cluster Issuer:

```
kubectl apply -f cert-manager-cluster-issuer.yaml
```

- If you have already traefik installed in kube-system namespace as k3s do make sure it has proper values:

```
helm upgrade traefik traefik/traefik -n kube-system --values traefik-values.yaml
```
