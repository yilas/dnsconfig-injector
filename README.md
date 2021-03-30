# Mutating Admission Webhook for dnsconfig pod injection


## Deploy

1. Create a signed cert/key pair and store it in a Kubernetes `secret` that will be consumed by dnsconfig deployment

```
./scripts/create-signed-cert.sh
```

2. Create the `MutatingWebhookConfiguration` by set `caBundle` with correct value from Kubernetes cluster

```
./scripts/patch-ca-bundle.sh
```

3. Deploy resources

```
kubectl apply -R -f k8s/

# #k8s/configmap.yaml  #custom DNSConfig
# apiVersion: v1
# kind: ConfigMap
# metadata:
#   name: dnsconfig-injector-webhook-configmap
#   namespace: kube-system
# data:
#   dnsconfig.yaml: |
#     nameservers:
#       - 1.2.3.4
#     searches:
#       - my.dns.search.suffix
#     options:
#       - name: ndots
#         value: "2"
#       - name: edns0
# dnsconfig-injector (master *)
```

## Verify

1. The dsnconfig inject webhook should be running
```
# kubectl -n kube-system get pods
NAME                                                  READY     STATUS    RESTARTS   AGE
dnsconfig-injector-webhook-deployment-bbb689d69-882dd   1/1       Running   0          5m
```

2. Label the default namespace with `dnsconfig-injector=enabled`
```
# kubectl label namespace default dnsconfig-injector=enabled
# kubectl get namespace --show-labels
NAME          STATUS    AGE       LABELS
default       Active    18h       dnsconfig-injector=enabled
kube-public   Active    18h
kube-system   Active    18h
```

3. Deploy an app in Kubernetes cluster, take `nginx` app as an example
```
# cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
EOF
```

4. Verify that the dnsconfig has been injected
```
# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7c97fff775-fbb8w   1/1     Running   0          36s

# kubectl exec -it nginx-7c97fff775-fbb8w -- cat /etc/resolv.conf
nameserver 10.100.200.10
nameserver 1.2.3.4
search default.svc.cluster.local svc.cluster.local cluster.local my.dns.search.suffix
options ndots:2 edns0
```
