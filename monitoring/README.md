# Monitoring instalation on k3s

## Preconditions

- Add Helm dependencies:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

- Create namespace monitoring

```
kubectl create namespace monitoring
```

- Create PersistentVolumeClaim for prometheus:

```
kubectl apply -f pvc-prometheus.yaml
```

## Instalation

- Install monitoring stack via Helm:

```
helm install kpstack prometheus-community/kube-prometheus-stack --namespace monitoring --values values.yaml
```

- To uninstall type following:

```
helm uninstall kpstack
```
