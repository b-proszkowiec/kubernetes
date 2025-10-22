# Jellyfin Installation on k3s

## Prerequisites

- Create the **jellyfin** namespace:

```
kubectl create namespace jellyfin
```

- Ensure Longhorn is installed to use its Persistent Volume Claims (PVCs)

- Create the certificate and IngressRoute:

```
kubectl apply -f jellyfin-certificate.yaml -f ingress-route.yaml
```

- Create the Longhorn PersistentVolumeClaim for Jellyfin:

```
kubectl apply -f longhorn-pvc-config.yaml
```

## Instalation

- Install the Jellyfin application via Helm:

```
helm install jellyfin-nas utkuozdemir/jellyfin -n jellyfin --values values.yaml
```

## Failover / Data Recovery

- To restore previous data (tested on Jellyfin version 10.10.7), copy the following files from your backup:

```
/config/config/system.xml
/config/config/users
/config/data/jellyfin.db
```
