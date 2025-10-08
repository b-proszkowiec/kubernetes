# Longhorn instalation on k3s

## Preconditions

- Create namespace longhorn-system

```
kubectl create namespace longhorn-system
```

- Install longhorn

```
helm install longhorn longhorn/longhorn -n longhorn-system
```

- Install ingressroute for longhorn

```
kubectl apply -f ingressroute.yaml
```
