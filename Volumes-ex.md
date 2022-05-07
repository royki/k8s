# Configure a volume for a Pod

- **EmptyDir**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

- **Host Path**

```yaml
apiVersion: v1
  kind: Pod
  metadata:
    name: test-ebs
  spec:
    containers:
    - image: k8s.gcr.io/test-webserver
      name: test-container
      volumeMounts:
      - name: test-data
        mountPath: /opt
    volumes:
    - name: test-data
      hostPath:
        path: /data
        type: Directory
```

- **Cloud Volumes**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: <volume-id>
      fsType: ext4
```

- Volume Name: pv-log
Storage: 100Mi
Access modes: ReadWriteMany
Host Path: /pv/log

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

- Volume Name: claim-log-1

```console
Storage Request: 50Mi
Access modes: ReadWriteOnce
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webapp
  name: webapp
spec:
  containers:
  - image: kodekloud/event-simulator
    name: webapp
    env:
      - name: LOG_HANDLERS
        value: file
    resources: {}
    volumeMounts:
      - name: container-log-data
        mountPath: /log
  volumes:
    - name: container-log-data
      persistentVolumeClaim:
        claimName: claim-log-1
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webapp
  name: webapp
spec:
  containers:
  - image: kodekloud/event-simulator
    name: webapp
    env:
      - name: LOG_HANDLERS
        value: file
    resources: {}
    volumeMounts:
      - name: container-log-data
        mountPath: /log
  volumes:
    - name: container-log-data
      hostPath:
        path: /var/log/webapp
        type: Directory
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
