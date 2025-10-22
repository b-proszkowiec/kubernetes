# NFS storage class instalation on k3s

## Preconditions

- Make sure NFS provisioner is installed and you have working share on external server:

```
kubectl get pods -A | grep nfs
```

- If not installed type following:

```
kubectl create ns nfs-csi
```

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
helm install nfs-csi nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --namespace nfs-csi \
    --set nfs.server=192.168.0.200 \
    --set nfs.path=/k3s_pv
```

- Install custom storage class to store PVC on NFS share:

```
kubectl apply -f nfs-storageclass.yaml
```
