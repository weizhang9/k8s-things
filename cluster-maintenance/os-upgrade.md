`kube-manager` waits 5m before consider the pod is dead and evict the pod, i.e. no more recovering. If the pod is not part of RS, then it's gone.

To upgrade OS, if you manage to get everything done under 5m, then you don't need to do anything for the pods. But most of the time this is not realistic.

A safer way to do so it to drain the node first, which will move all the pods to other available nodes
```
k drain <node name>
```
When you run `drain` on a node, the node is also marked as `cordon`ed so no pods can be scheduled onto it.

After upgrade and reboot, need to uncordon the node so that pods can be scheduled on it again.
```
k uncordon <node name>
```

Cordon a node won't make sure the pods on it would get moved to other pods, but will just ensure no more pods can be scheduled onto it.
```
k cordon <node name>
```
