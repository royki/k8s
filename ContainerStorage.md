# Container Storage

Images are made from stackable layers which help to deploy changes quickly and effectively.
The image layers combine with a writeable top container layer along with other read-only image layers to make up what we call the container.
When a container is removed, it is the top writeable layer that is being removed leaving behind the underlying image layers.
As a result of using copy-on-write for the top writeable layer, when the container is removed, associated data is lost.

In order to have persistent data with containers, the data needs to be externalized and persisted outside of the container and not applied as a container layer.

1. A bind mount or mounts for short is a file or folder stored anywhere on the container host file system and mounted into a running container.
When using a mount, you need to set the mount flag when running a new container and specify the source folder on the container host.
Mounts, can actually be modified by processes outside of the container management.
Mounts do provide an easy way to access the underlying files from the host, but data is not initialized and it is common to run into permission issues.

2. Volumes are stored by the host file system as well, but they are completely managed by the container agent.
Volumes utilize the mount flag to specify the volume to be mounted to a new container.
(But instead of specifying a local directory, you specify the name of the volume that you want to mount to the container.)
Volumes still make it easy to access persistent data on the underlying host, but permission issues are far less common and volumes provide the additional capability to access non-local storage systems such as NFS mounts.

Amazon Elastic Block Store volumes or EBS for your underlying host storage, the EBS volumes are specific to availability zones
and cannot be mounted across AZs or to multiple Elastic Compute Cloud instances at the same time.
If trying to share volume access across containers, the containers would need to be on the same underlying instance or the volume would need to be cloned using an EBS Snapshot
and then creating a new volume.
