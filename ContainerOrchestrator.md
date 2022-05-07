# Container

- The benefits to using containers are numerous, particularly with Docker.
- Docker images can run on many different platforms. If you can run an application on a Linux kernel, then you can package it in Docker, and then run it in different environments without having to change it.
- Containers keep the concerns of development and operations separate. What happens inside the container doesn’t have much impact on operations that run outside the container. So, applications can do their own thing, while the process of running the container can be standardized.
- Containers are easy to manage – you can quickly create and delete, download, share, start and stop them.
- Containers use hardware resources more efficiently than virtual machines, and are more lightweight. They are more dense than virtual machines. Docker containers can share layers, and support multi-tenancy.
- Containers should be treated as unchangeable, so that every container created from a single image will be the same. Container images are versions. You should not patch a container, but patch the image when it needs to be updated. When you don’t need it anymore, a container can be easily deleted or replaced.

## Most container orchestrators can

- Group hosts together while creating a cluster
- Schedule containers to run on hosts in the cluster based on resources availability
- Enable containers in a cluster to communicate with each other regardless of the host they are deployed to in the cluster
- Bind containers and storage resources
- Group sets of similar containers and bind them to load-balancing constructs to simplify access to containerized applications by - creating a level of abstraction between the containers and the user
- Manage and optimize resource usage
- Allow for implementation of policies to secure access to applications running inside containers.

## Container orchestrators are tools which group systems together to form clusters where containers' deployment and management is automated at scale while meeting the requirements mentioned below

- Fault-tolerance
- On-demand scalability
- Optimal resource usage
- Auto-discovery and communicate with each other
- Accessibility from the outside world
- Seamless updates/rollbacks without any downtime.

### Container Orchestrator (Kubernetes, Docker Swarm)

- A Container Orchestrator manages the deployment, placement & lifecycle of workload containers.
- It can federate multiple compute hosts or nodes into one target called a cluster.
- It distributes containers across nodes.
- It distributes client requests across containers.
- It discover services.
- It replicates nodes and containers to handle increased workload.
- It replaces unhealthy nodes & containes.

### Container Ecosystem

- Layer 1 is the physical infrastructure, the actual hardware where resources are deployed. This hardware might be owned, or leased through the cloud.
- Layer 2 is the virtualization of that infrastructure, through the use of software like VMWare, or Amazon Web Services.
- Layer 3 is the operating system. You can choose one or more operating systems to deploy containers, including Ubuntu, RedHat, CoreOS, and others.
- Layer 4 is the container engine that manages running containers on the operating system, and might include container image management, or just raw container services. Docker is a container engine.
- Layer 5 is for container scheduling. This layer manages the lifecycle of containers, including starting, stopping, and destroying them as required. This function is covered by some container orchestrators, like Kubernetes.
- Layer 6 is for container orchestration. This layer enforces the defined requirements for a set of containers, does scheduling to enforce those requirements, and offers services for monitoring container health and logging statistics. Kubernetes has all these features.
- Layer 7 is the application workflow. It supports the creation of container images from source code, identifies and configures service dependencies, routing, and service provisioning by using the container orchestration layer.

### Container Scheduling (Apache Swarm)
