# Network Policy

- Scenario: Web POD(Port 80), API POD(Port 5000), DB POD(Port 3306). All the PODs can communicate to each other with the kubernetes cluseter.
- Implement a Network Policy to restrict the access between Web POD to DB POD. i.e.
  - Web POD can only communicate with API POD but not with DB POD
  - API POD can communicate with both API POD and DB POD (same as before)
  - DB POD can only communicate with API POD not with Web POD
- Network Policy is an object in Kubernetes like POD, ReplicaSet etc

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress  
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api-pod
    ports:
    - protocol: TCP
      port: 3306  
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: iron-gallery-firewall
spec:
  podSelector:
    matchLabels:
      db: mariadb
      run: iron-gallery
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          db: mariadb
          run: iron-gallery
    ports:
    - protocol: TCP
      port: 3306
```

- `kubectl create -f np-ingress.yaml`

- By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace. The following examples let you change the default behavior in that namespace.
- Create a “default” isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any ingress traffic to those pods.

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

- Allow all ingress traffic to all pods in a namespace (even if policies are added that cause some pods to be treated as “isolated”)

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```

- Deny all egress traffic

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

- Allow all egress traffic

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-Egress
spec:
  podSelector: {}
  egress:
  - {}
  policyTypes:
  - Egress
```

- Deny all ingress and all egress traffic

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## Another Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

- Not all Network Interface support Network Policies.
- Network Interface like Kube-router, Calico, Romana, Weave-net supports Network Policy
- Flannel doesn't support network policy

### Test the Network-policy

- `kubectl run --generator=run-pod/v1 test-np --image=busybox:1.28 --rm -it -- sh nc -z -v -w 2 ServiceName Port`
- `kubectl run --generator=run-pod/v1 test-np --image=busybox:1.28 --rm -it -- sh` (Enter to the container)
- Run into the container
  - `nc -z -v -w 2 ServiceName Port`
  