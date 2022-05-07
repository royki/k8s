# Container Network Interface

- Container Runtime must create network namespace
- Identify network the container must attach to
- Container Runtime to invoke Network Plugin (bridge) when container is ADDed
- Container Runtime to invoke Network Plugin (bridge) when container is DELeted
- JSON format of the Network Configuration
- Must support command line arguments ADD/DEL/CHECK
- Must support parameters container id, network ns etc..
- Must manage IP Address assignment to PODs
- Must Return results in a specific format

## Configuring CNI

- Kubelet is installed via Kubeadm/minikube
  - `cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
- Kubelet is installed as binary
  - `kubelet.service`

- CNI plugin type -

```console
--network-plugin=cni
--cni-bin-dir=/opt/cni/bin
--cni-conf-dir=/etc/cni/net.d
```

- CNI is found at
  - `ls /opt/cni/bin`
  - `ls /etc/cni/net.d`

- `cat /etc/cni/net.d/10-bridge.conf`

```json
{
  "cniVersion": 0.2.0,
  "name": "mynet",
  "type": "bridge",
  "isGateway": true,
  "isMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.22.0.0/16",
    "routes": [
      {
        "dst": "0.0.0.0/0"
      }
    ]
  }
}
```

- Deploy Weave
- `kubectl apply -f "http://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n'")`
  - `deamonset.extensions/weave-net created`
- Weave Peers via Kubeadm tool as pod in `kube-system` namespace.
