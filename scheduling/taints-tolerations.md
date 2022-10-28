Taints are used on the nodes and tolerations used on the pods.

<span style="color:blue">Taints</span> specify the special conditions of the <span style="color:blue">nodes</span>, <span style="color:green">tolerations</span> specify the conditions the <span style="color:green">pods</span> can stand. By default, pods have no tolerations. Without taints or tolerations, k8s scheduler will spread out the pods equally on the nodes.

They are both used to __restrict if a node can take certain pods__. If a pod has __tolerations that match the taints__ on a node, then the pod can be scheduled onto the node. HOWEVER the pod can be scheduled on any other nodes without any taints too. If you want to restrict a pod to a specific node, then we need to use `Affinity` instead.

- to specify taint on node
```bash
k taint nodes <node-name> <taint-key>=<taint-value>:<taint-effect>
# k taint nodes node01 app=frontend:NoSchedule
```  
there are 3 taint-effect: `NoSchedule`, `PreferNoSchedule`, `NoExecute`(pods after taints defined won't scheduled w/o the matching toleration, and existing pods on the node will be evicted if w/o matching toleration)

- to specify tolerations on pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-app
spec:
  containers:
    - name: frontend
      image: frontend:v1
  tolerations: # all fields need quoting
    - key: "app"
      operator: "Equal"
      value: "frontend"
      effect: "NoSchedule"
```

A typical use case for this is the `kubemaster` node. Upon creation, k8s taints its own master cluster to make sure no pods are scheduled onto the master node. This can be viewed at:
```bash
k describe node kubemaster | grep Taint
# Taints:   node-role.kubernetes.io/master:NoSchedule
```
