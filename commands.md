### ETCD

`etcdctl` is the cli tool interacting with etcd, it has v2 and v3, by default it's set to use v2. Set `export ETCDCTL_API=3` to use v3

v2 vs v3 examples:

| v2  | v3  |
|-----|-----|
| etcdctl backup | etcdctl snapshot save |
| etcdctl cluster-health | etcdctl endpoint health |
| etcdctl mk | etcdctl get |
| etcdctl mkdir | etcdctl get |
| etcdctl set | etcdctl put |

You must also specify path to certificate files so that `etcdctl` can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path:
```
--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key
```

- Example command to get all keys from k8s etcd:
```bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```

- view kube-api options in a cluster  
  -- using kubeadm
    ```bash
    # k8s deploy kube-api in a `kubet-apiserver-master` node in kube-system namespace
    kubectl get pods -n kube-system
    # you can then view the kube-api options in the kube-api node
    cat /etc/kubernetes/manifests/kube-apiserver.yaml
    ```
  -- manual boostrap
    ```bash
    # see the option from service config
    cat /etc/systemd/system/kube-apiserver.service
    # list the running process and effected options to it
    ps -aux | grep kube-apiserver
    ```

- view kube-controller-manager options in a cluster  
  -- using kubeadm
    ```bash
    # k8s deploy kube-controller-manager in a `kubet-controller-manager-master` node in kube-system namespace
    kubectl get pods -n kube-system
    # you can then view the kube-controller-manager options in the node
    cat /etc/kubernetes/manifests/kube-controller-manager.yaml
    ```
  -- manual boostrap
    ```bash
    # see the option from service config
    cat /etc/systemd/system/kube-controller-manager.service
    # list the running process and effected options to it
    ps -aux | grep kube-controller-manager
    ```

- view kube-scheduler options in a cluster  
  -- using kubeadm
    ```bash
    # k8s deploy kube-scheduler in a `kubet-scheduler-master` node in kube-system namespace
    kubectl get pods -n kube-system
    # you can then view the kube-scheduler options in the node
    cat /etc/kubernetes/manifests/kube-scheduler.yaml
    ```
  -- manual boostrap
    ```bash
    # see the option from service config
    cat /etc/systemd/system/kube-scheduler.service
    # list the running process and effected options to it
    ps -aux | grep kube-scheduler
    ```

- view kubelet options in a cluster:
    ```bash
    # list the running process and effected options to it
    ps -aux | grep kubelet
    ```

### Exam Tips
- use `k run` to create **pod** yaml
```bash
# generate redis image pod manifest yaml and save it to pod.yaml without creating the pod (the --dry-run bit)
k run redis --image=redis --dry-run=client -o yaml > pod.yaml
```
```bash
# same as above but output the generated yaml in stdout
k run nginx --image=nginx --dry-run=client -o yaml
```

- use `k run` to create **pod** directly instead of composing yaml and apply it
```bash
k run nginx --image=nginx
```

- use `k create` to create a **deployment** directly instead of composing yaml and apply it
```bash
k create deployment --image=nginx nginx
```

- use `k create` to generate **deployment** yaml (with `--replicas` if k8s >= v1.19)
```bash
k create deployment --image=nginx nginx (--replicas=4) \
 --dry-run=client -o yaml > deployment.yaml
```

- use `k edit` to apply change to resource yaml on the fly
```bash
kubectl edit pod <pod-name>
```

- create a ClusterIP service  
__√ this one will use pod's label as selector__
```bash
# create a `redis-service` ClusterIP service expose pod redis on port 6379
k expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml
```
__this one doesn't allow selector, so if the pod has different labels, generate the yaml first and edit the selector, then apply instead__
```bash
# this will assume selectors as app=redis
k create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

- create a NodePort service  
__√ this one will use pod's label as selector but you can't specify a nodePort, so generate yaml and edit nodePort if needed__
```bash
k expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
__this one won't use pod's label as selector__
```bash
k create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
