- create a pod in a defined name with a port exposed
```bash
k run my-nginx-pod --image nginx --port 8080
```

- create a pod with image, then create a cluster ip svc to expose the pod on a target port
```bash
k run my-nginx-pod --image nginx --port 8080 --expose
```
...