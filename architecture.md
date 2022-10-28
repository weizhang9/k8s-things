- how kube-api works when crud a service in k8s
![](./graph/update-service-process.png)

- manually bootstrap kube-api, install k8s the hard way (kubeadm provides this out of box)
![](./graph/install-kubeapi-manually.png)

- kube controllers watch and remediate the state
![](./graph/kube-controllers.png)

- manually bootstrap kube-controller-manager
![](./graph/install-kubecontrollermanager-manually.png)

- kube scheduker decide which pod goes where based on various criteria set, and kubelet on each node creates the pod
![](./graph/scheduler-example.png)

- manually bootstrap kube-scheduler
![](./graph/install-kubescheduler-manually.png)

- kubelet: captain on worker nodes, creates pods; single POC for master node, sending heartbeats to master node
![](./graph/kubelet-task.png)

- manually install kubelets on worker nodes *even when using kubeadm*
![](./graph/manually-install-kubelet-on-worker-nodes.png)

- kube proxy is a process running on each node, it looks for new services and create rules to forward traffic from services to its pod. services is just a virtual component only lives in k8s's memory with no interface, it's accessible by all nodes because of kube proxy
![](./graph/kube-proxy-route-traffic-thru-iptable.png)

- manually install kube proxy and run it as a service
![](./graph/manually-install-kubeproxy.png)

- kubeadm deploys kube proxy as a daemonset
![](./graph/kube-proxy-deployed-as-daemonset.png)
