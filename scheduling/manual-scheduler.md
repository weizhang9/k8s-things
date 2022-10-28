K8s has scheduler to schedule a pod to a suitable node. If a pod is not scheduled, it will remain in `pending` state.

You can manually schedule a pod __at creation time__ by adding a `nodeName` to pod's spec in manifest, e.g.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  nodeName: node02 # <-- usually this field is filled by k8s automatically
```

If you want to schedule the pod to a node after the pod is created, you won't be able to do it thru manifest, but can use pod's binding API to mimic it, e.g.
```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```
then convert the above yaml to json, and post it to the pod's binding API as request body, e.g.
```bash
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind":"Binding", ...}' \
https://${SERVER}/api/v1/namespaces/${NS}/pods/${PODNAME}/binding/
```
