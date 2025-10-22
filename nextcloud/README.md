# Nextcloud instalation on k3s

## Preconditions

- Create the **nextcloud** namespace:

```
kubectl create namespace nextcloud
```

- Make sure NFS provisioner is installed. If not install it from **nfs-storage** section.

```
kubectl get pods -A | grep nfs
```

- Install required PVC:

```
kubectl apply -f nfs-pvc-app.yaml -f nfs-pvc-db.yaml
```

- Create db secret to prevent root password change in case of files migration:

```
kubectl apply -f db-secret.yaml
```

## Instalation

- Install the Nextcloud application via Helm:

```
helm install nextcloud-nas nextcloud/nextcloud -n nextcloud --values values.yaml
```

- To uninstall type following:

```
helm uninstall nextcloud-nas
```

## Ingress route

- Type following to enable traefik ingress:

```
kubectl apply -f nextcloud-certificate.yaml
kubectl apply -f ingress-route.yaml
```

## Additional tips

- If application stack while creating **nextcloud-nas-redis-replicas-0** try to restart one of the node.
