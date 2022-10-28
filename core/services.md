Services enable connectivity between pods within and outside of the same cluster, and with external apps.

### NodePort service
It listens to a port on the Node and forward requests on that port to a port on the Pod running the application inside the Node.  This can happen to 1) one pod on one node 2) multiple pods on one node 3) multiple pods on multiple nodes.
- The port on the Pod is called `TargetPort`.
- The port on the Service is called `Port`, as these ports are defined from the service's POV. The service inside the Node is like a  VM, it has its own IP, which is called  `ClusterIP` of the service.
- The port on the Node is called `NodePort`, this is the port we use to access the  internal Pod externally through the Node. `NodePort` has a default range of 30,000 - 32,767.
```yaml
# example node port service manifest
apiVersion: v1
kind: Service
metadata:
    name: frontend
spec:
    type: NodePort
    ports: # array type, so can have multiple port mappings
      - targetPort: 80 # if omitted, will be set same as port
        port: 80
        nodePort: 30008 # if omitted, will assign randomly within the range
    selector: # use the pod's label here to link service to pods
      tier: frontend
      app: my-app
```

### ClusterIP service
It creates a virtual IP for the cluster to enable communication between different services such as frontend servers to a set of backend servers. This is because the automatically assigned pod IPs are not persistent.  
ClusterIP is a persistent abstraction layer that provides a single interface to connect to these pods. After creation, the service can then be access through cluster IP or service name, e.g. `<service-name>.<ns>.svc.cluster.local`
```yaml
# example cluster ip service manifest
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP # this is the default Service type if not specified type
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: my-app
    tier: backend
```

### LoadBalancer service
When NodePort service is used for multiple pods living on multiple nodes, you will get multiple endpoints to access each node (and their underlying pods). This is tedious and unpractical, so we need LB. LoadBalancer service type provisions an LB for our service in *supported cloud providers* like GCP, AWS, Azure. If created in unsupported environment, it will just create a NodePort service.
```yaml
# example load balancer service manifest
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    tier: frontend
    app: my-app
```
