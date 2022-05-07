# DNS in Kubernetes

- Kubernetes set a DNS server by default, when it is installed.
- How DNS in Kubernetes helps PODs to resolve other PODs and services within the Cluster.
- When a service is created, the Kubernetes DNS server creates a IP record for the service. It maps the newly created service name to that IP address.
So within the cluster any POD can reach the service using its sevice name, within same namespace. `curl http://service_name`
- *If the service is in another namespace, to access the service, we need to add `.namespace` at the end. `curl http://service_name.namespace`
- For each namespace DNS server creates a sub-domain.
- All the services are grouped together into another sub-domain called `svc`.
- ALl the services and pods are grouped together into root-domain for the cluster which is set to `cluster.local` by default.
- `service-name(service hostname) <- namespace <- type(svc) <-  root doamin(root) <- IP Address`  [Generic]
  - `curl http://service_name.namespace.svc.cluster.local` [Fully Qualified Domain Name(fqdn) for the service], *root-domain is always `cluster.local`*
- Records for PODs are not created by default. But we can enable that explicitly.
- For each POD, Kubernetes generated a name by replacing the dot(.) with dash(-).
- For POD, the Fully Qualified Domain Name -
  - `service-name(service hostname) <- namespace <- type(pod) <-  root doamin(root) <- IP Address`  [Generic]
  - `curl http://10-244-2-5.namespace.pod.cluster.local` [fqdn]

## Ingress

- Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
- Ingress Controller & Ingress Resources
- `kubectl get ingress`

## Ingress Resources

- Ingress rules
  - Each HTTP rule contains the following information:
    - An optional host. In this example, no host is specified, so the rule applies to all inbound HTTP traffic through the IP address specified. If a host is provided (for example, foo.bar.com), the rules apply to that host.
    - A list of paths (for example, /testpath), each of which has an associated backend defined with a serviceName and servicePort. Both the host and path must match the content of an incoming request before the load balancer directs traffic to the referenced Service.
    - A backend is a combination of Service and port names as described in the Service doc. HTTP (and HTTPS) requests to the Ingress that matches the host and path of the rule are sent to the listed backend.
- A default backend is often configured in an Ingress controller to service any requests that do not match a path in the spec.
  - Path Types
    - `ImplementationSpecific` (default): With this path type, matching is up to the `IngressClass`.
    - `Exact`: Matches the URL path exactly and with case sensitivity.
    - `Prefix`: Matches based on a URL path prefix split by `/`. Matching is case sensitive and done on a path element by element basis.

- Ingress Class

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: IngressClass
metadata:
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com/v1alpha
    kind: IngressParameters
    name: external-lb
```

- Single Service Ingress

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80
```

- PS: You can also do this with an Ingress by specifying a default backend with no rules.

- Fanout: A fanout configuration routes traffic from a single IP address to more than one Service, based on the HTTP URI being requested. An Ingress allows you to keep the number of load balancers down to a minimum.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: simple-fanout-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: service1
          servicePort: 4200
      - path: /bar
        backend:
          serviceName: service2
          servicePort: 8080
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: iron-gallery-ingress
spec:
  rules:
  - host: iron-gallery-braavos.com
    http:
      paths:
      - path: /
        backend:
          serviceName: iron-gallery-service
          servicePort: 80
```

- Name-based virtual: Name-based virtual hosts support routing HTTP traffic to multiple host names at the same IP address.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: service1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: service2
          servicePort: 80
  - http:
      paths:
      - backend:
          serviceName: service3
          servicePort: 80
```

- PS: the following Ingress resource will route traffic requested for first.bar.com to service1, second.foo.com to service2, and any traffic to the IP address without a hostname defined in request (that is, without a request header being presented) to service3
