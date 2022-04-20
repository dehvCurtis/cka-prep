# Storage [10%]

Understand storage classes, persistent volumes

Table of Contents

- StorageClasses
- [Persistent Volume](#Persistent-Volume)
  - [Configure Pod to Use PersistentVolume for Storage](#Configure-Pod-to-Use-PersistentVolume-for-Storage)

## Persistent Volume

https://kubernetes.io/docs/concepts/storage/persistent-volumes/

### Configure Pod to Use `PersistentVolume` for Storage

Summary

1. You, as cluster administrator, create a `PersistentVolume` backed by physical storage. You do not associate the volume with any Pod.
2. You, now taking the role of a developer / cluster user, create a `PersistentVolumeClaim` that is automatically bound to a suitable PersistentVolume.
3. You create a Pod that uses the above `PersistentVolumeClaim` for storage

**Create an index.html file on your Node[ ](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-an-index-html-file-on-your-node)**

on node (or local), create a `/mnt/data` directory:

```shell
sudo mkdir /mnt/data
```

In the `/mnt/data` directory, create an `index.html` file:

```shell
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
```

Test that the `index.html` file exists:

```shell
cat /mnt/data/index.html
```

The output should be:

```
Hello from Kubernetes storage
```

Close the shell to your Node (if not on local).

**Create a PersistentVolume**

In this exercise, you create a *hostPath* PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

In a production cluster, you would not use hostPath. Instead a cluster administrator would provision a network resource like a Google Compute Engine persistent disk, an NFS share, or an Amazon Elastic Block Store volume. Cluster administrators can also use [StorageClasses](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#storageclass-v1-storage) to set up [dynamic provisioning](https://kubernetes.io/blog/2016/10/dynamic-provisioning-and-storage-in-kubernetes).

Create config `pv-volume.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Create the `PersistentVolume`:

```shell
kubectl apply -f pv-volume.yaml
```

View information about the PersistentVolume:

```shell
kubectl get pv task-pv-volume
```

The output shows that the PersistentVolume has a `STATUS` of `Available`. This means it has not yet been bound to a PersistentVolumeClaim.

```
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO           Retain          Available             manual                   4s
```

**Create a PersistentVolumeClaim**

Create yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

Create the PersistentVolumeClaim:

```
kubectl apply -f pv-claim.yaml
```

After you create the PersistentVolumeClaim, the Kubernetes control plane looks for a PersistentVolume that satisfies the claim's requirements. If the control plane finds a suitable PersistentVolume with the same StorageClass, it binds the claim to the volume.

Look again at the `PersistentVolume`:

```shell
kubectl get pv task-pv-volume
```

Now the output shows a `STATUS` of `Bound`.

```
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                   STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO           Retain          Bound     default/task-pv-claim   manual                   2m
```

Look at the `PersistentVolumeClaim`:

```shell
kubectl get pvc task-pv-claim
```

The output shows that the PersistentVolumeClaim is bound to your PersistentVolume, `task-pv-volume`.

```
NAME            STATUS    VOLUME           CAPACITY   ACCESSMODES   STORAGECLASS   AGE
task-pv-claim   Bound     task-pv-volume   10Gi       RWO           manual         30s
```

**Create a Pod**

Create a Pod that uses your `PersistentVolumeClaim` as a volume.

Create `pv-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

Create the Pod

```shell
kubectl apply -f pv-pod.yaml
```

Verify that the container in the Pod is running;

```shell
kubectl get pod task-pv-pod
```

Get a shell to the container running in your Pod:

```shell
kubectl exec -it task-pv-pod -- /bin/bash
```

In your shell, verify that nginx is serving the `index.html` file from the hostPath volume:

```shell
apt update
curl http://localhost/
```

The output shows the text that you wrote to the `index.html` file on the hostPath volume:

```
Hello from Kubernetes storage
```