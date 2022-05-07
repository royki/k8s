# Docker Private Registry

- `docker login private-registry.io`
- `kubectl create secret docker-registry regcred --docker-server=private-registry.io --docker-username=xxxx --docker-password=xxxx --docker-email=xxx@yyy.com`

## Container Security

- `docker run --user=1001 ubuntu sleep 3000`
- `docker run --cap-add ADMIN ubuntu`
- Congifure same in Kubernetes
  - In Kubernetes containers are encapsulated in PODs.
  - The security settings can be configured at Container Level or POD Level.
  - If the securtiy settings are configured at POD level, then the security will be applied to all containers within the POD.
  - If the securtiy settings are configured at POD & Container level, then the Container security settings will overwrite the POD security settings.
  - Setting the security at POD level

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: pod-nginx
    name: pod-nginx
  spec:
    securityContext:
      runAsUser: 1000
    containers:
    - image: nginx
      name: pod-nginx
      resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
  status: {}
  ```

  - Security at Container Level

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: pod-nginx
    name: pod-nginx
  spec:
    containers:
    - image: nginx
      name: pod-nginx
      securityContext:
        runAsUser: 1000
        capabilities: # Capabilities are only supported at the container level not the POD level
          add: ["Admin"]
      resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
  status: {}
  ```
