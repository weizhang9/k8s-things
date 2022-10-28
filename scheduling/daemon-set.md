DaemonSet is like ReplicaSet in the way that it runs multiple duplicated pods on different nodes, but DaemonSet will run **one** copy of the pod on **each** node. If a node is removed, the pod on it is removed too from DaemonSet. This is perfect for functionalities like metric collecting or networking. `kube-proxy` is a DaemonSet.

DaemonSet manifest is similar to ReplicaSet:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent
```

A quick way to create DaemonSet is to use `kubectl` to create a deployment with dry run to generate yaml first, then change the `kind` to `DaemonSet` in yaml and delete `Strategy`, `Replicas` and `Status` in yaml, then apply the yaml to create the DaemonSet:
```bash
k create deploy <deploy-name> --image=nginx -o yaml --dry-run=client > ds.yaml
```

Create the DaemonSet with
```bash
k create -f daemonset.yaml
```

View the DaemonSet with
```bash
k get ds
```

View a specific DaemonSet with
```bash
k describe ds monitoring-daemon
```

Before k8s v1.12, DaemonSet uses `nodeName` field on each pod to assign a pod to each node.
Since k8s v1.12, DaemonSet uses node affinity and default scheduler to achieve it.
