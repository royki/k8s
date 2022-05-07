# API Groups in Kubernetes

- K8S APIs are grouped by in multiple ways - `/version`, `/apis`, `/api`, `/metrics`, `/healthz`, `logs` etc.
- APIs responsible for cluster functionalities - core groups: `/api`, named groups: `/apis`
- core grouped -> `/api` -> `/v1` -> (`/namespaces`, `/pods`, `/rc`, `events`, `/endpoints`, `/nodes`, `/bindings`, `/pv`, `/pvc`, `/configmaps`, `/secrets`, `/services`) etc.
- named groups -> `/apis` -> (`/apps`, `/extensions`, `/networking.k8s.io`, `/authentication.k8s.io`, `/certificates.k8s.io`) etc

- `curl https://kube-master:6443/version`
- `curl https://kube-master:6443/api/v1/pods`
- `curl http://localhost:6443 -k --key admin.key --cert admin.crt --cacert ca.crt`
- `kubectl proxy` -> is a http control proxy service created by `kubectl` to access the `kube-api` server.
  - `Starting to serve on 127.0.0.1:8001`
