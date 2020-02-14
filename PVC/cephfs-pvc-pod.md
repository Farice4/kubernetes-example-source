### crate pvc

> display pvc
[root@k8s-node1 ~]# kubectl get pvc task-pv-claim 
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
task-pv-claim   Bound    pvc-f9362575-4c93-11ea-9ea7-fa163e72cd4a   1Gi        RWO            cephfs         3d4h

> display pv
[root@k8s-node1 ~]# kubectl describe pv pvc-f9362575-4c93-11ea-9ea7-fa163e72cd4a
Name:            pvc-f9362575-4c93-11ea-9ea7-fa163e72cd4a
Labels:          <none>
Annotations:     cephFSProvisionerIdentity: cephfs-provisioner-1
                 cephShare: kubernetes-dynamic-pvc-f9a21c78-4c93-11ea-b8a0-0242ac113603
                 pv.kubernetes.io/provisioned-by: ceph.com/cephfs
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    cephfs
Status:          Bound
Claim:           default/task-pv-claim
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:        CephFS (a CephFS mount on the host that shares a pod's lifetime)
    Monitors:    [10.1.29.48:6789 10.1.29.49:6789 10.1.29.50:6789]
    Path:        /volumes/kubernetes/kubernetes/kubernetes-dynamic-pvc-f9a21c78-4c93-11ea-b8a0-0242ac113603
    User:        kubernetes-dynamic-user-f9a21cc0-4c93-11ea-b8a0-0242ac113603
    SecretFile:  
    SecretRef:   &SecretReference{Name:ceph-kubernetes-dynamic-user-f9a21cc0-4c93-11ea-b8a0-0242ac113603-secret,Namespace:default,}
    ReadOnly:    false
Events:          <none>


### Pod
创建的Pod使用cephfs方式，path使用上面查看到的pv Path路径

> Create Pod
"""
kind: Pod
apiVersion: v1
metadata:
  namespace: admin
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      cephfs:
        monitors: 
        - 10.1.29.48:6789
        - 10.1.29.49:6789
        - 10.1.29.50:6789
        path: /volumes/kubernetes/kubernetes/kubernetes-dynamic-pvc-f9a21c78-4c93-11ea-b8a0-0242ac113603
        user: "admin"
        secretRef:
          name: ceph-secret
  containers:
    - name: task-pv-container
      image: nginx
      resources:
        requests:
          memory: 1Gi
          cpu: 300m
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage

"""

> List Pod
[root@k8s-node1 ~]# kubectl get pods -n admin
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   0          6m58s

> display Pod
[root@k8s-node1 ~]# kubectl describe pods -n admin task-pv-pod 
Name:               task-pv-pod
Namespace:          admin
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node1/10.1.32.111
Start Time:         Fri, 14 Feb 2020 17:59:03 +0800
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"task-pv-pod","namespace":"admin"},"spec":{"containers":[{"image":"ngi...
Status:             Running
IP:                 172.17.54.16
Containers:
  task-pv-container:
    Container ID:   docker://4633a4d0f1943faabfada5e8be911e44fb4a803f9ed837a86ec3cf8ea8d4e8c9
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:ad5552c786f128e389a0263104ae39f3d3c7895579d45ae716f528185b36bc6f
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 14 Feb 2020 17:59:20 +0800
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        300m
      memory:     1Gi
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from task-pv-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-swr8k (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  task-pv-storage:
    Type:        CephFS (a CephFS mount on the host that shares a pod's lifetime)
    Monitors:    [10.1.29.48:6789 10.1.29.49:6789 10.1.29.50:6789]
    Path:        /volumes/kubernetes/kubernetes/kubernetes-dynamic-pvc-f9a21c78-4c93-11ea-b8a0-0242ac113603
    User:        admin
    SecretFile:  
    SecretRef:   &LocalObjectReference{Name:ceph-secret,}
    ReadOnly:    false
  default-token-swr8k:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-swr8k
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s


> Login pod
[root@k8s-node1 ~]# kubectl exec -it -n admin task-pv-pod bash
root@task-pv-pod:/# df -Th /usr/share/nginx/html/
Filesystem                                                                                                                                 Type  Size  Used Avail Use% Mounted on
10.1.29.48:6789,10.1.29.49:6789,10.1.29.50:6789:/volumes/kubernetes/kubernetes/kubernetes-dynamic-pvc-f9a21c78-4c93-11ea-b8a0-0242ac113603 ceph   14T  967G   13T   8% /usr/share/nginx/html


