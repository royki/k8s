# Kubernetes Volumes

- Volume: Has a lifetime, and can outlast a container, thereby preserving data across container restarts.
A Kubernetes Volume is a directory that is accessible by containers in a Pod.

- emptyDir: An entity that is created when a Pod is assigned to a Node.
The emptyDir exists while that Pod is running on that Node. As its name indicates, it is initially empty.
When a Pod is removed from a Node for any reason, the data in the emptyDir is deleted forever.

- NFS: A Volume that allows an NFS (Network File System) share to be mounted into your Pod.
Unlike emptyDir, which is erased when a Pod is removed, the contents of an NFS volume are preserved and the Volume is unmounted.
An NFS Volume can be prepopulated with data and that data can be moved between Pods.
NFS can be mounted by multiple writers simultaneously.

## The Kubernetes Volume abstraction solves the following problems

- Pods (containers) are ephemeral. If the container crashes, the data are lost.
- If running multiple containers in a Pod, those containers might need to share data.
- Docker also has a concept of Volume, but it is simply a directory on a disk or in a container.

- A Pod can specify what Volumes are needed (the `.spec.volumes` field) and where to mount those into Containers(the `.spec.containers.volumeMounts` field).
- Volumes can not mount onto other Volumes or have hard links to other Volumes.
- Each Container in the Pod must independently specify where to mount each Volume.
