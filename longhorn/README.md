# Longhorn instalation on k3s

## Preconditions

- Go to official instruction and run script to check if your nodes supports longhorn:

  https://longhorn.io/docs/1.10.0/deploy/install/#checking-prerequisites-using-longhorn-command-line-tool

- Create namespace longhorn-system

```
kubectl create namespace longhorn-system
```

## Instalation

- Install the Longhorn application via Helm:

```
helm install longhorn longhorn/longhorn -n longhorn-system --values values.yaml
```

- Enable daily backup:

```
kubectl apply -f daily-backup.yaml
```

## Ingress route

- Check if you already set ingress in the values.yaml. If not run:

```
kubectl apply -f ingressroute.yaml
```
