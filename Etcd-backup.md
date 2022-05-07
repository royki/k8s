# Backup etcd server

- `etcd.service`

```console
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --
  --
  ...
  ...
  --data-dir=/var/lib/etcd
```

- ETCD built-in snapshot tools
- Taking a snapshot of etcd

```console
ETCDCTL_API=3 etcdctl snapshot save etcd-snapshot.db \
                    --endpoints=https://127.0.0.1:2379 \
                    --cacert=/etc/etcd/ca.crt \
                    --cert=/etc/etcd/etcd-server.crt \
                    --key=/etc/etcd/etcd-server-key
```

- Check the status of etcd backup/snapshot
  - `ETCDCTL_API=3 etcdctl snapshot status etcd-snapshot.db`
- Restore from etcd backup/snapshot
  - Stop etcd service
    - `service etcd.service stop`
    - `service kube-apiserver stop`

    ```console
    ETCDCTL_API=3 etcdctl \
      snapshot restore etcd-snapshot.db \
      --data-dir /var/lib/etcd-from-backup \
      --initial-cluster master-1=https://192.168.5.11:2380,master-1=https://192.168.5.11:2381 \
      --initial-cluster-token etcd-cluster-1 \
      --initial-advertise-peer-urls https://${INTERNAL_IP}: 2380
    ```

    - *a new backup directory is created along with a new token*

    ```console
    **etcd.service**
      ExecStart=/usr/local/bin/etcd \\
      --name ${ETCD_NAME} \\
      --
      --
      ...
      --initial-cluster-token etcd-cluster-1
      --data-dir=/var/lib/etcd-from-backup
    ```

    - `systemctl daemon-reload`
    - `service etcd restart`
    - service kube-apiserver start`

- Export `etcd` version `$ export ETCDCTL_API=3`
- `etcdctl snapshot save -h`
- `etcdctl snapshot restore -h`
- `--cacert` - verify certificates of TLS-enabled secure servers using this CA bundle
- `--cert` -  identify secure client using this TLS certificate file
- `--endpoints=[127.0.0.1:2379]` - This is the default as ETCD is running on master node and exposed on localhost 2379
- `--key` - identify secure client using this TLS key file
