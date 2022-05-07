# Imperative Command

- Deploy a nginx pod using the nginx:alpine image with the labels set to tier=db

```shell
kubectl run --image=nginx:alpine nginx-pod --dry-run=client -o yaml > nginx-pod.yaml
kubectl run --image=nginx:alpine nginx-pod --port=8080 --expose --dry-run=client -o yaml > nginx-pod-service.yaml
kubectl apply -f nginx-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
spec:
  containers:
  - image: nginx:alpine
    name: nginx-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Deploy a redis pod using the redis:alpine image with the labels set to tier=db. Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.

```shell
kubectl run --image=redis:alpine redis --labels=tier=db  --dry-run=client -o yaml > redis-pod.yaml
kubectl apply -f redis-pod.yaml
```

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    tier: db
  name: redis
spec:
  containers:
  - image: redis:alpine
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Create a service redis-service to expose the redis application within the cluster on port 6379. Use imperative commands.
  - Type: ClusterIP, Port: 6376

```shell
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP --dry-run=client -o yaml > redis-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    tier: db
  name: redis-service
spec:
  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    tier: db
  type: ClusterIP
status:
  loadBalancer: {}
```

- Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas. Try to use imperative commands only. Do not create definition files.
  - Name: webapp
  - Image: kodekloud/webapp-color
  - Replicas: 3

```shell
kubectl create deployment --image=kodekloud/webapp-color webapp --replicas=3 --dry-run=client -o yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: kodekloud/webapp-color
        name: webapp-color
        resources: {}
status: {}
```

- Create a new pod called custom-nginx using the nginx image and expose it on container port 8080

```shell
kubectl run --image=nginx custom-nginx --dry-run=client -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: custom-nginx
  name: custom-nginx
spec:
  containers:
  - image: nginx
    name: custom-nginx
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Create a new namespace called dev-ns. Use imperative commands.

```shell
kubectl create namespace dev-ns
```

- Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas. Use imperative commands.

```shell
 kubectl create deployment --image=redis redis-deploy --namespace=dev-ns --replicas=2 --dry-run=client -o yaml
 kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: redis-deploy
  name: redis-deploy
  namespace: dev-ns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: redis-deploy
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}
```

- Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80. Try to do this with as few steps as possible.
  - 'httpd' pod created with the correct image?
  - 'httpd' service is of type 'ClusterIP'?
  - 'httpd' service uses correct target port 80?
  - 'httpd' service exposes the 'httpd' pod?

```shell
kubectl run --image=httpd:alpine httpd --port=80 --dry-run=client -o yaml
kubectl expose pod httpd --type=ClusterIP --port=80 --dry-run=client -o yaml
#  OR
kubectl run httpd --image=httpd:alpine --port=80 --expose
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: httpd
  name: httpd
spec:
  containers:
  - image: httpd:alpine
    name: httpd
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: httpd
  name: httpd
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: httpd
  type: ClusterIP
status:
  loadBalancer: {}
```
