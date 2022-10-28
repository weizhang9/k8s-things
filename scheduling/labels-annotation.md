### Labels
Labels are used to group k8s objects by type, function, app, etc.

We can use labels to filter object, e.g.
```bash
# selector will use the kv pair specified under `labels` to filter
k get pods --selector app=App1
```

K8s itself uses labels to connect different objects (types of resources) together, e.g.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-webapp
  labels: # <-- this is RS's labels, used by other objects to discover RS
    app: App1
    function: frontend
spec:
  replicas: 3
  selector: # <-- this is selector used by RS to connect to pods
    matchLabels:
      app: App1
  template:
    metadata:
      labels: # <-- this is pod's labels, used by RS to discover pods
        app: App1
        function: frontend
    spec:
      containers:
        - name: my-webapp
          image: my-webapp
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-sevice
spec:
  selector:
    app: App1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```


### Annotations
Annotations, also under the `metadata` field, are used to record other information like app/build version, tool names, contact info, etc, which can be used for some integration purpose.
