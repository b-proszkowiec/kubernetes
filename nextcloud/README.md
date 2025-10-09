# Nextcloud instalation on k3s

## Preconditions

- Create the **jellyfin** namespace:

```
kubectl create namespace jellyfin
```

- Make sure NFS provisioner is installed:

```
    kubectl get pods -A | grep nfs
```

- If not install by Typing following:

```
    kubectl create ns nfs-csi
    kubectl create ns nextcloud
```

```
    helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
    helm repo update
    nfs.csi.k8s.io
    helm install nfs-csi nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
        --namespace nfs-csi \
        --set nfs.server=192.168.0.200 \
        --set nfs.path=/nextcload_pv
```

- Install custom storage class to store PVC on NFS share:

```
    kubectl apply -f nfs-storageclass.yaml
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

- Install the Jellyfin application via Helm:

```
   helm install nextcloud-nas nextcloud/nextcloud -n nextcloud --values values.yaml
```

- To uninstall type following:

```
  helm uninstall nextcloud-nas
```
