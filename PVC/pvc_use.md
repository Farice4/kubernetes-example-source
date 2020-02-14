This is about ceph backend pvc use record.

# StorageClass
Creating storageclass before pvc create.

### Storage Create
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  creationTimestamp: 2018-11-19T02:07:42Z
  name: rook-ceph-block
  resourceVersion: "224835"
  selfLink: /apis/storage.k8s.io/v1/storageclasses/rook-ceph-block
  uid: e5e32372-eb9f-11e8-a04e-52540006a3fe
parameters:
  clusterNamespace: rook-ceph
  fstype: xfs
  pool: replicapool
provisioner: ceph.rook.io/block
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

### PVC Create

> Create local pvc

```
kind: PersistentVolume
apiVersion: v1
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

> Create storageclass pvc

'''
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
'''

### Pod Create

```
kind: Pod
apiVersion: v1
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


