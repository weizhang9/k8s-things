One of the 2 ways to assign a node to a pod/cluster, e.g. when a pod/cluster requires more resources and you have a larger node dedicated to this.

There are 2 steps to achieve this:
1. we need to label the node
```bash
k label nodes <node-name> <label-key>=<label-value>
```

2. add `nodeSelector` to manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
spec:
    containers:
    - name: data-processor
      image: data-processor
    nodeSelector:
      size: Large # so the node would "size=Large" label
```

Limitation of node selector is that it can only contains simple logic using matching label, if we want something more complex, like `!small` or `large || medium`. Node Affinity to the rescue!
