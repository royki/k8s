# Upgrade Kubernetes API Version Using Kubeadm tool

- `kube-apiserver(X)` is the primary component in the Control Plane, hence none of the other component will be higher version than `kube-apiserver`
  - `X = v1.10`
- `Controller-manager(X-1)`, `kube-scheduler(X-1)` can be 1 version lower.
  - `X = v1.10` or `v1.09`
- `kubelet(X-2)`, `kube-proxy(X-2)` can be 2 version lower.
  - `X = v1.10` or `v1.09` or `v1.08`
- `kubelet` version can be higher, same or lower than the `kube-apiserver`
  - `X = v1.11` or `v1.10` or `v1.09`
- At anytime Kubernetes supports upto recent 3 minor versions.
- We can upgrade one minor version at a time.
- The recommneded approach to upgrade version is to upgrade one minor api version at a time. `v1.10` -> `v1.11` -> `v1.12`.
- `kubeadm upgrade plan`
- `kubeadm upgrade apply`
- Upgrading a cluster involving 2 major steps - 1st Upgrade the Master Node 2nd Upgrade the Worker Nodes.
- During upgradation of Master Node, the Control Plane (API Server, Scheduler, Controller Managers) goes dwn briefly.
  - The Worker Nodes remains up as well as the applications.
  - As Master Node is down, the management control is down i.e. there will not be any new scheduling for POD or Deployments or if any POD goes down, it will not be up automatically.
  - The Nodes are not accessable by `kubectl` or other Kubernets API.
- Upgrade Worker Nodes can be done differently.
- Upgrade all worker nodes at a single time. In this way the applications will be down and user ll have an impact. It requires a down time.
- Upgrade worker nodes one by one. In this way the applications will not be down and there is no down time.
- Upgrade worker nodes by adding new nodes with newer API version to the cluster.(Convenient in Cloud).

## Kubeadm Command

- `kubeadm upgrade plan`
  - ***Kubeadm doesn't install or upgrade kubelet.***
  - ***kubelet API needs to be upgraded manually in each worker node after kubeadm api upgrade operation***.
- `kubeadm upgrade apply API_VERSION`

## Upgrade API Version on Master Node

- **Before performing kubeadm upgradation, kudeadm itself needs to be upgraded.**
  - `apt upgrade -y kubeadm=1.12.0-00`
  - Apply Upgrade by `kubeadm`
  - `kubeadm upgrade apply v1.12.0`
  - **Master Node may or may not have *kubelet*, it depends on installation process.**
  - In this stage `kubectl get nodes` command ll still display the older version becuase the version of kubelet on master node is registered with apiserver and not the version of the apiserver itself. (`kubectl` is still in older version.) So need to update the `kubelet` on Master Node.
  - `apt upgrade -y kubelet=1.12.0-00`
  - `systemctl restart kubelet`

### Upgrade API Version on Worker Nodes

- Upgrade worker nodes one by one
  - `kubectl drain node-01` (This will cordon as well as evicts all the PODs to other Nodes)
  - `apt upgrade -y kubeadm=1.12.0-00`
  - `apt upgrade -y kubelet=1.12.0-00`
  - `kubeadm upgrade node config --kubelet-version v1.12.0`
  - `systemctl restart kubelet`
  - `kubectl uncordon node-01`
