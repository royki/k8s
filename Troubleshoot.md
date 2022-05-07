# Stacked Topology

- External ETCD Topology
- `apiserver` is only component that talks to the etcd-server. So we need to mention the etcd-server ip in apiserver config file. i.e apiserver is pointing the right address of etcd-servers.
- `cat /etc/systemd/system/kube-apiserver.service` -> `-- etcd-server=http://10.240.10:2379, https://10.240.11:2379`
- `etcd.service`
- `export ETCDCTL_API=3`
- `etcdctl put key value`
- `etcdctl get key`
- `etcdctl get / --prefix --keys-only`

- majority/quorum = (n/2 -1) [n = no of instances in a cluster]
- Min no of instances in a High Availability Cluster is 3.
- Odd no of instances is preferrable over even no of instances in a cluster.
- Having even number of instances can leave the cluster without quorum in certain network partition scenarios.
- kubeconfig files are used by clients to access the `kube-apiserver`. The admin user, controller-manager, scheduler, kubeproxy are all clients to the kube-api server. So they
all can user kubeconfig files to communicate with the `kube-apiserver`.
- kubeconfig files for `kube-proxy` which will run on the worker nodes, the kube-controller-manager, scheduler & the admin user.
- The components that live with the `kube-apiserver` on the same master nodes, such as the `controller-manager` & `scheduler` can reach the `api-server` directly at the loopback address of `127.0.0.1`.
- Those outside the master nodes, such as the `admin user`, the `kube-proxy` reach the `api-server` through the load balancer.

## Troubleshooting

- Application Failure
  - Check if Application is accessable
    - `curl http://web-service-ip:node-port`
  - Check Service port, target-port, selector name(with pod/deployment)
    - `kubectl describe service service_name`
  - Check Pod
    - `kubectl get pods -owide`
    - `kubectl describe pod pod_name`
  - Check Logs
    - `kubectl logs pod_name`
    - `kubectl logs pod_name -f --previous`
    - Check Service Logs

- Control Plane Failure
  - Check Node Status
    - `kubectl get nodes -owide`
    - `kubectl get pods -owide`
    - `kubectl -n kube-system get pods -owide`
  - Check Component Status
    - `service kube-apiserver status`
    - `service kube-controller-manager status`
    - `service kube-scheduler status`
  - Check on Worker Node
    - `service kubelet status`
    - `service kube-proxy status`
  - Check Service Logs
    - `kubectl -n kube-system logs kube-apiserver-master`
    - `sudo journalctl -u kube-apiserver`
  - Check Kubelet(How it is deployed - Static Pod or )
    - `cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
      - `Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"`

- Search For StaticPod Configuration File
  - `grep -i staticPodPath /var/lib/kubelet/config.yaml`
    - `/etc/kubernetes/manifests`
    - `ls -al`
- Worker Node Failure
  - Check Node Status
    - `kubectl get nodes -owide`
    - `kubectl describe node node_name`
  - Check Service & Port running on Worker Node
    - `top/htop`
  - Check the storage capacity on Nodes
    - `df -h`
  - Check Kubelet status
    - `service kubelet status`
    - `sudo journalctl -u kubelet`
  - Check Certificate on Worker Nodes
    - `openssl x509 -in /var/lib/kubelet/worker-1.crt -text`
    - `unable to load client CA file: unable to load client CA file: open /etc/kubernetes/pki/ca.crt: no such file or directory`

### Json Path output in Kubernetes

- `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'`
  - `master node01`
- `kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'`
  - `amd64 amd64`
- `kubectl get nodes -o=jsonpath='{.items[*].status.cpu}'`
  - `4 4`
- `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{.items[*].status.cpu}'`
  - `master node01 4 4`
- `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.cpu}'` \\ {"\t"}

    ```console
    master node01
    4  4
    ```

- Loops/Range
  - `kubectl get nodes -o=jsonpath='{range.items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'`

- Custom Colummns
  - `kubectl get nodes -o=custom-columns=<COLUMN NAME>:<JSON PATH>`
  - `kubectl get nodes -o=custom-columns=NODE:.metadata.name`
  - `kubectl get nodes -o=custom-columns=NODE:.metadata.name, CPU:.status.capacity.cpu`

- Sorting JSON Path
  - `kubectl get nodes --sort-by= .metadata.name`
  - `kubectl get nodes --sort-by= .status.capacity.cpu`

- `kubectl get nodes -ojson > /opt/outputs/nodes.json`
- `kubectl get nodes -ojson > /opt/outputs/nodes.json`
- `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"} > /opt/outputs/node_names.txt`
- `kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}{"\n"}'`
- `kubectl get nodes -o=jsonpath='{range.items[*]}{.status.nodeInfo.osImage}{end}{"\n"}'`
- `kubectl get nodes -o=jsonpath='{range.items[*]}{.status.nodeInfo.osImage}{"\n"}{end}' > /opt/outputs/nodes_os.txt`
- `kubectl config view --kubeconfig=/root/my-kube-config -o=jsonpath='{.users[*].name}{"\n"}{.contexts[*].name}{"\n"}{.contexts[*].context.user}{"\n"}' > /opt/outputs/users.txt`
- `kubectl get pv --sort-by=.spec.capacity.storage`
- `kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage`
- `kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name`
