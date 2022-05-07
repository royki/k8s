# Persistent Volumes & Persistent Volumes Claim

- PVs are resources in the cluster. PVCs are requests for those resources and also act as claim checks to the resource
- The interaction between PV & PVC follows the lifecycle
  - Provisioning
    - Static:
    - Dynamic:
  - Binding:
  - Using:
  - Storage Object in Use Protection:
  - Reclaiming
    - Retain:
    - Delete:
    - Recycle:
      - **Warning: (`k8sV1.18`) The Recycle reclaim policy is deprecated. Instead, the recommended approach is to use dynamic provisioning.**
- **PersistentVolumes binds are exclusive, and since PersistentVolumeClaims are namespaced objects, mounting claims with “Many” modes (`ROX, RWX`) is only possible within one namespace**
- **(`k8sV1.18`) - Currently, a PVC with a non-empty selector can’t have a PV dynamically provisioned for it.**

## PV Volume Mode

- Kubernetes supports two `volumeModes` of PersistentVolumes: `Filesystem` & `Block`
- `volumeMode` is an optional API parameter. `Filesystem` is the default mode used when `volumeMode` parameter is omitted.
- A volume with `volumeMode: Filesystem` is mounted into Pods into a directory.
- If the volume is backed by a block device and the device is empty, Kuberneretes creates a filesystem on the device before mounting it for the first time.

## PV Access Modes

- A PersistentVolume can be mounted on a host in any way supported by the resource provider.
- The access modes are:
  - `ReadWriteOnce(RWO)` – the volume can be mounted as read-write by a single node.
  - `ReadOnlyMany(ROX)` – the volume can be mounted read-only by many nodes.
  - `ReadWriteMany(RWX)` – the volume can be mounted as read-write by many nodes.
- **A volume can only be mounted using one access mode at a time, even if it supports many.**

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

### awsElasticBlockStore

- Creating an EBS volume
  - `aws ec2 create-volume --availability-zone=eu-west-1a --size=10 --volume-type=gp2`

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

### PV Definition File

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol-1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

- `kubectl create -f pv-definition.yaml`
- `kubectl get persistentvolume`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```

### PersistentVolumeClaim (make the storage available in the node)

- Admin create the set of PVs in the nodes & developers/users create PVC to user those PVs.
- Once the PVC is created, K8s binds the PVCs with PVs appropriately.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
```

- `kubectl create -f pvc-definition.yaml`
- `kubectl get persistentvolumeclaim`

- Claims as Volumes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: block-pvc
```

- PersistentVolume using a Raw Block Volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```

- PersistentVolumeClaim requesting a Raw Block Volume

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```

- Pod specification adding Raw Block Device path in container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
```
