Deployment is a higher level abstraction of replicaset and pods.

Deployment can do deploy, upgrade, rolling update, pause and resume deployment.

### Commands
- create deployment
```bash
k create -f deployment-definition.yaml
```

- view deployment
```bash
# show you the deployment with the deployment's name set in metadata
k get deployments
```

- view replicaset
```bash
# show you the replicaset with the deployment's name
k get rs
```

- view pods
```bash
# show you the pods with the deployment's name
k get pods
```

- view all of them (deployment, rs, pods)
```bash
k get all
```
