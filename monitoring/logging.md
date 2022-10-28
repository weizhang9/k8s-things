k8s's logging is pretty similar to docker, i.e.
```bash
# add -f to tail the logs
k logs -f <pod-name> [<container-name-if-multiple-container>]
```
