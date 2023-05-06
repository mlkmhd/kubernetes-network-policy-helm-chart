This repository helps you define Kubernetes network policies in a more readable and easier syntax.

here is the `values.yaml` file that you can define your network policies:
```
ingress:

  crds:
  - name: client
    host: 192.168.0.0/16
    port: 8080
    
  - name: client
    host: 192.168.0.0/16
    port: 8443
    
  - name: www-https
    host: 192.168.1.1/32
    port: 8443

  pods:

  - name: prometheus
    namespace: monitoring
    labels:
      - app.kubernetes.io/name: prometheus
    port: 8080
    protocol: TCP

egress:
  
  crds:
  
  - name: logstash
    host: 192.168.1.1/32
    port: 30310
  - name: redis
    host: 192.168.1.1/32
    port: 30353

  pods:
    
  - name: kube-dns
    namespace: kube-system
    labels:
      - k8s-app: kube-dns
    port: 53
    protocol: UDP
```

you can install this helm chart using the following command: (In the example above, I assumed that you want to apply a `deny all` policy for all pods in the `test` namespace and then open some ingress and egress ports. For instance, this manifest file enables all pods within the `test` namespace to connect to the kube-dns pod in the `kube-system` namespace.)

```bash
$ helm install my-network-policy . -n test
NAME: my-network-policy
LAST DEPLOYED: Sat May  6 14:39:46 2023
NAMESPACE: test
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: network-policy/templates/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
  namespace: test
spec:
  policyTypes:
    - Ingress
    - Egress

  podSelector:
    matchLabels: {}

  ingress:
    - from:
        - podSelector: {}
    - from:
        - ipBlock:
            cidr: 192.168.0.0/16
      ports:
        - port: 8080
    - from:
        - ipBlock:
            cidr: 192.168.0.0/16
      ports:
        - port: 8443
    - from:
        - ipBlock:
            cidr: 192.168.1.1/32
      ports:
        - port: 8443
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: monitoring
          podSelector:
            matchLabels:
              app.kubernetes.io/name: prometheus
      ports:
        - port: 8080
          protocol: TCP

  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - port: 53
          protocol: UDP
    - to:
        - ipBlock:
            cidr: 192.168.1.1/32
      ports:
        - port: 30310
    - to:
        - ipBlock:
            cidr: 192.168.1.1/32
      ports:
        - port: 30353

    - to:
        - podSelector: {}
```
