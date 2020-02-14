### kubernetes support pod mount pvc

Example: admin namespace create admin endpoint, create pvc in default namespace, create pod in admin namespace and mount pvc in default namespace

> create endpoint

endpoint.json
"""
{
    "kind": "Endpoints", 
    "subsets": [
        {
            "addresses": [
                {
                    "ip": "192.168.250.161"
                }
            ], 
            "ports": [
                {
                    "port": 1
                }
            ]
        }, 
        {
            "addresses": [
                {
                    "ip": "192.168.250.162"
                }
            ], 
            "ports": [
                {
                    "port": 1
                }
            ]
        }, 
        {
            "addresses": [
                {
                    "ip": "192.168.250.163"
                }
            ], 
            "ports": [
                {
                    "port": 1
                }
            ]
        }
    ], 
    "apiVersion": "v1", 
    "metadata": {
        "name": "admin"
    }
}

"""

kubectl apply -f endpoint.json -n admin

> create pvc
test-pvc.yaml
"""
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: chenyb-test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
"""

kubectl apply -f test-pvc.yaml

> create pod

test-pod.yaml
"""
kind: Pod
apiVersion: v1
metadata:
  name: task-gfs-pvc
  namespace: admin
spec:
  volumes:
    - glusterfs:
        endpoints: admin
        path: vol_1c7e11a74c9d79188ce8cc8d2a6e9465
      name: task-pv-storage
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

kubectl apply -f test-pod.yaml 

> Display pod

"""
[root@k8s-node1 ~]# kubectl describe pods -n admin task-gfs-pvc 
Name:               task-gfs-pvc
Namespace:          admin
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node2/192.168.250.162
Start Time:         Fri, 14 Feb 2020 10:47:55 +0800
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"task-gfs-pvc","namespace":"admin"},"spec":{"containers":[{"image":"ng...
Status:             Running
IP:                 172.17.75.15
Containers:
  task-pv-container:
    Container ID:   docker://3bb5890987f2fb120ab3f3327ae4472a717111693e0ad69a4eb9b27edc34fb42
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:ad5552c786f128e389a0263104ae39f3d3c7895579d45ae716f528185b36bc6f
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 14 Feb 2020 10:48:22 +0800
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        300m
      memory:     1Gi
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from task-pv-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-g6qdn (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  task-pv-storage:
    Type:           Glusterfs (a Glusterfs mount on the host that shares a pod's lifetime)
    EndpointsName:  admin
    Path:           vol_1c7e11a74c9d79188ce8cc8d2a6e9465
    ReadOnly:       false
  default-token-g6qdn:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-g6qdn
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s

"""
