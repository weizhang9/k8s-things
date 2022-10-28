There are 3 namespaces created on start automatically by k8s, namely:  
`default` - all the resources user created live by default if not specified  
`kube-system` - store all k8s system settings like DNS, they are stored in separate ns to avoid getting deleted by accident  
`kube-public` - store all services that should be available to all users


### Namespace DNS
You can connect to service within same ns simply by referring the service name, e.g.
```
mysql.connect("db-service")
```

To connect to service in a different ns, refer it as `<service-name>.<ns>.svc.cluster.local`, e.g.
```
mysql.connect("db-service.dev.svc.cluster.local")
```

`cluster.local` is the default cluster domain name

`svc` is the subdomain for services

`dev` is the namespace

`db-service` is the service name

### Create resources in different namespace
- by command
```bash
k create -f pod-definition.yaml -n=mynamespace
```
- by manifest by adding `namespace` field in metadata
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    namespace: dev
    labels:
      app: myapp
      tier: frontend
spec:
    containers:
    - name: nginx-container
      image: nginx
```

### Namespace creation
- by command
```bash
k create namespace <mynamespace>
```
- by manifest
```yaml
apiVersion: v1
kind: Namespace
metadata:
    name: <mynamespace>
```

### Create resource quota for a namespace
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: <mynamespace>
spec:
    hard:
      pods: "10"
      requests.cpu: "4"
      requests.memory: 5Gi
      limits.cpu: "10"
      limits.memory: 10Gi
```
