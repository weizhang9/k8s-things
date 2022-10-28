Node Affinity ensures pods get scheduled onto the matching node with complex logic. Take the same large pod to large node example like node selector, the manifest file would become:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In # NotIn # Exists
              values:
                - Large
                - Medium
              # - Small
```

NodeAffinity has two types
1. `requiredDuringSchedulingIgnoredDuringExecution`
2. `preferredDuringSchedulingIgnoredDuringExecution`

You can use the operator field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt` and `Lt`.

You can also use Node Affinity or Anti-affinity to control which pods can coexist on a node.

`NotIn` and `DoesNotExist` allow you to define node anti-affinity behavior. Alternatively, you can use node taints to repel Pods from specific nodes.

Note:
- If you specify both `nodeSelector` and `nodeAffinity`, **both** must be satisfied for the Pod to be scheduled onto a node.

- If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the Pod can be scheduled onto a node if **one** of the specified `nodeSelectorTerms` can be satisfied.

- If you specify multiple `matchExpressions` associated with a single `nodeSelectorTerms`, then the Pod can be scheduled onto a node only if **all** the `matchExpressions` are satisfied.

Inter-pod affinity and anti-affinity: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
