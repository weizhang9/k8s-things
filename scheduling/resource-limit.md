By default k8s will set limit of 1 CPU and 512 Mi memory on containers, you can change this in manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    name: my-app
spec:
  containers:
    - name: my-app
      image: my-app
      ports:
        - containerPort: 8080
  resources:
    request:
      memory: "1Gi"
      cpu: 1
    limits:
      memory: "2Gi"
      cpu: 2
```

0.1 CPU is 100milli CPU, CPU can go as low as 1m.

1 CPU = 1000m CPU = 1 AWS vCPU = 1 GCP Core = 1 Azure Core = 1 Hyperthread.

Memory can be specified by bytes, kilobyte or kibibyte.
1 K (kilobyte) = 1000 bytes
1 Ki (kibibyte) = 1024 bytes

If a node tries to consume more CPU than available, the pod will throttle.
If a node tries to consume more memory than available, the pod will be`OOMKilled`.

If you want a pod to init with default cpu and memory, you can set `LimitRange` in the namespace where the pods will be created:
```yaml
# limit range for mem
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range-my-ns
spec:
  limits:
    - default:
        memory: 512Mi
      defaultRequest:
        memory: 256Mi
      type: Container
---
# limit range for cpu
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range-my-ns
spec:
  limits:
    - default:
        cpu: 1
      defaultRequest:
        cpu: 0.5
      type: Container
```

**NOTE**
For a running pod, you cannot edit its resource limits[1]. You can:
1. If the pod is not part of a deployment, you can edit the pod with `kubectl edit pod <pod name>`, which won't apply the change but will save a copy of your modified manifest to a tmp location, which you can then apply after deleted the existing running pod.
2. If the pod is not part of a deployment, you can export the pod manifest with `kubectl get pod webapp -o yaml > my-new-pod.yaml`, make the needed change, and then delete and apply new manifest like first option.
3. If the pod is part of a deployment, you can simply edit the deployment with `kubectl edit deployment <deployment-name>`, as pod template is a child of the deployment, once you saved the edited deployment, the deployment will automatically rollout the new change for you.
