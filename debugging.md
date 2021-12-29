# Kubernetes Debugging

<!-- `make toc` to generate https://github.com/jonschlinkert/markdown-toc#cli -->

<!-- toc -->

- [Resource Management](#resource-management)
    + [list node capacity](#list-node-capacity)
    + [get pod spec](#get-pod-spec)
    + [get resource requests and limits (for a pod if specified)](#get-resource-requests-and-limits-for-a-pod-if-specified)
    + [check resource consumption for the nodes/pods](#check-resource-consumption-for-the-nodespods)
- [Pod Security Policy](#pod-security-policy)
    + [list (cluster) roles and associated resources](#list-cluster-roles-and-associated-resources)
    + [find out which (cluster) roles are associated with the PSP](#find-out-which-cluster-roles-are-associated-with-the-psp)
    + [list (cluster) role bindings for ServiceAccounts](#list-cluster-role-bindings-for-serviceaccounts)
- [Kubernetes Network Model](#kubernetes-network-model)
    + [check cluster network info](#check-cluster-network-info)
    + [list the cidr range of nodes](#list-the-cidr-range-of-nodes)
    + [log into a node](#log-into-a-node)
    + [deploy network debugging tool](#deploy-network-debugging-tool)
    + [debug pod unable to resolve network service](#debug-pod-unable-to-resolve-network-service)
    + [slow performance](#slow-performance)
      - [bonus - perfomance test with curl](#bonus---perfomance-test-with-curl)
- [Multi-platform Kubernetes Clusters](#multi-platform-kubernetes-clusters)
    + [get the taints associated with the nodes](#get-the-taints-associated-with-the-nodes)
    + [display nodes labels](#display-nodes-labels)
    + [check pod logs if describe pod isn't helpful](#check-pod-logs-if-describe-pod-isnt-helpful)
- [Cluster Management](#cluster-management)
    + [view deployment details](#view-deployment-details)
    + [view resources details](#view-resources-details)
    + [modify resources thru cli](#modify-resources-thru-cli)
    + [problem with mixing imperative and declarative](#problem-with-mixing-imperative-and-declarative)
- [Kubernetes Deployment](#kubernetes-deployment)
    + [StatefulSet vs ReplicaSet vs Deployment vs Service vs DaemonSet vs Job vs Rolling Deployment](#statefulset-vs-replicaset-vs-deployment-vs-service-vs-daemonset-vs-job-vs-rolling-deployment)
    + [rolling deployment commands](#rolling-deployment-commands)
    + [deployment scale](#deployment-scale)
- [kubernetes Logging](#kubernetes-logging)
    + [kubectl logs flags](#kubectl-logs-flags)
    + [check logs on the node](#check-logs-on-the-node)
- [Kubernetes Monitoring & Alerting](#kubernetes-monitoring--alerting)
- [Eviction](#eviction)

<!-- tocstop -->

## [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container)
Specify resource requirements via requests and limits in the Pod spec
1. request: minimum quantity of the resource needed
2. limits: maximum allowed usage for the specified resource

Kubernetes scheduler ensures that, for each resource type, the *sum* of the resource requests by all scheduled pods on the node is less than the capacity of the *node*. For example, the CPU resource's actual usage on the pods might be low but the scheduler will still refuse to create a pod on the node if the capacity check failed.

#### list node capacity
```
kubectl describe node ${node-name}
```

#### get pod spec
```
kubectl get ${pod-name} -n ${namespace} -o yaml
```

#### get resource requests and limits (for a pod if specified)
```
kubectl get pods ${pod-name} -o=jsonpath='{.spec.containers[0].resources}'
```

#### check resource consumption for the nodes/pods
```
kubectl top ${node|pods}
```

## [Pod Security Policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)
Requirements for using PSP
1. create a pod security policy
2. create a cluster role
3. create a cluter role binding for namespace and subject (user, service account, etc)

#### list (cluster) roles and associated resources
```
kubectl get roles,clusterroles --all-namespaces -o custom-columns='KIND:kind,NAMESPACE:metadata.namespace,NAME:metadata.name,RESOURCES:..resources'
```

#### find out which (cluster) roles are associated with the PSP
```
kubectl get roles,clusterroles --all-namespaces -o custom-columns='KIND:kind,NAMESPACE:metadata.namespace,NAME:metadata.name,RESOURCES:..resources' | grep -i podsecuritypolicies
```

#### list (cluster) role bindings for ServiceAccounts
```
kubectl get rolebingdings,clusterrolebindings \
--all-namespaces \
-o custom-columns='KIND:kind,NAMESPACE:metadata.namespace,NAME:metadata.name,SERVICE_ACCOUNTS:subjects[?(@.kind=="ServiceAccount")].name'
```

## [Kubernetes Network Model](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
1. every pod gets its own IP addr
2. pods on nodes can communicate with *all* the pods on *all* the nodes without NAT
3. agents on a node can communicate with *all* the pods on *that node*

#### check cluster network info
```
kubectl cluster-info
```

#### list the cidr range of nodes
```
kubectl get nodes -o jsonpath='{range .items[*]} {.metadata.name}{"   "}{.spec.podCIDR}{"\n"}{end}'
``` 

#### log into a node
```
# get the node internal ip from node info first
kubectl get nodes -o wide
# ssh into the node
ssh ${user}@${internal-ip}
# display route table in the node
route -n
```

#### deploy network debugging tool
```
# deploy a debugging tool image
...
kind: Deployment
...
spec:
	template:
	...
		spec:
			containers:
			- name: network-multitool-deploy
		  	  image: praqma/network-multitool
				...

# capture packet info - have to capture from host network as pods have no access to network interface
kind: DaemonSet
...
spec:
	template:
	...
		spec:
			hostNetwork: true
			...
			
# bash into debug tool container and run debug commands
kubectl exec -it ${network-debugging-tool-container} bash
# example network debug commands
ip link show
route -n # will list each pod's network interface
tcpdump -i ${interface-name} -vvv
```

#### debug pod unable to resolve network service
```
# deploy network debug tool and bash into the pod
nslookup kubernetes.default # check default dns
cat /etc/resolv.conf # check if dns can be resolved

# check if dns is running ok
kubectl get pods -n ${namespace} | grep -i dns
# check ip forwarding is working (can be turned off after a security scan)
ssh ${user}@${node-internal-ip} # login to the node
sysctl -a | grep ip_forward # check ip forwarding
sysctl -w net.ipv4.ip_forward=1 # enable ip forwarding (not persistent)
# if ip forwarding is on, there might be other firewall issues, debug with tcpdump https://sslhow.com/tcpdump-tcp/
```

#### slow performance
```
# login to node
netstat -i # check if packets behaviour suspicious, e.g. high TX-DRP
netstat -s | grep -i fragment # check fragment transmission failure
# set MTU correctly e.g.
# restart node if changed network settings
```

##### bonus - perfomance test with curl
```
# curl-format.txt
 time_namelookup: %{time_namelookup}\n
       time_connect: %{time_connect}\n
 time_appconnect: %{time_appconnect}\n
  time_pretransfer: %{time_pretransfer}\n
       time_redirect: %{time_redirect}\n
time_starttransfer: %{time_starttransfer}\n
						----------------\n
			time_total: %{time_total}\n
-------------------------------------------------------\n
   size_download: %{size_download}\n
       size_header: %{size_header}\n
      size_request: %{size_request}\n
       size_upload: %{size_upload}\n
speed_download: %{speed_download}\n
    speed_upload: %{speed_upload}\n
```
usage:
```
curl -w "@curl-format.txt" -o /dev/null -s ${URL_TO_CURL}
```

## Multi-platform Kubernetes Clusters
1. Use [Docker image manifests](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/) to handle multi-arch and multi-OS images
2. Use Kubernetes [scheduling primitives](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/), e.g. node selector, node affinity, taints and tolerations (allow nodes to repel pods if pods not tolerate nodes' taints), for targeted deployment of workloads in a mixed-platform cluster.

#### get the taints associated with the nodes
```
kubectl get nodes -o jsonpath='{range .items[*]} {.metadata.name}{"    "}{.spec.taints}{"\n"}{end}'
```

#### display nodes labels
```
kubectl get nodes --show-labels # some labels are added by k8s
```

#### check pod logs if describe pod isn't helpful
```
kubectl describe pod ${pod-name}
kubectl logs ${pod-name}
```

## Cluster Management
- k8s dashboard
-- https://github.com/kubernetes/dashboard
-- https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
- [k8s cli](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)
-- requires config file under ${HOME}/.kube
-- specify the config file via `KUBECONFIG` env var or `--kubeconfig` flag
-- https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
-- [kubectl autocomplete](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#optional-kubectl-configurations-and-plugins)
-- https://swissarmydevops.com/containers/kubernetes/kubernetes-cheat-sheet
-- https://kui.tools/

#### view deployment details
```
kubectl create -f ${manifest} # imperative approach - tells what to do - useful for delete
kubectl apply -f ${manifest} # declarative approach - tells what you want - useful for create
# get deployments
kubectl get deploy
# view last applied deployment config
kubectl apply view-last-applied deployment/${deployment-name} 
kubectl get deployment/${deployment-name} -o yaml # any config changed is added to the annotations field
```

#### view resources details
```
# get services
kubectl get svc
kubectl get svc --sort-by=.metadata.name # kubectl get svc ${svc-name} to see which field you want to use
# get pods
kubectl get pods --selector=app=nginx # use whatever selector you want, can get selector from deployment config
```

#### modify resources thru cli
all the on the fly changes are not saved unless used with `--save-config` flag
```
kubectl edit deployment ${deployment-name} # edit deployment config on the fly, not used for prod
kubectl scale --replicas=2 deployment ${deployment-name} # scale up and down on the fly
kubectl set image deployment ${deployment-name} ${deployment-name}=${new-image} # change deployment image on the fly
kubectl rollout status -w deployment ${deployment-name} # rollout the change above while viewing deployment status
kubectl rollout undo deployment ${deployment-name} # undo the change above
kubectl patch deploy ${deployment-name} -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx-sample","env":[{"name":"RESTART","value":"'$(date +%s)'"}]}]}}}}' # add env var on the fly
kubectl patch deploy ${deployment-name} --patch '{"spec":{"template":{"spec":{"containers":[{"name":"${app-name}","resources":{"limits":{"memory":"300M"}}}]}}}}' # change resource limit on the fly
```

#### problem with mixing imperative and declarative
```
# imperative to scale the deployment on the fly
kubectl scale deployment ${deployment-name} --replicas=2
# change deployment image from config file then rollout
kubectl replace -f ${deployment-config-file} # pod scale is overridden
kubectl apply -f ${deployment-config-file} # pod scale remains
```

## Kubernetes Deployment

#### StatefulSet vs ReplicaSet vs Deployment vs Service vs DaemonSet vs Job vs Rolling Deployment
Deployment and StatefulSet are abstraction of k8s workload management, which should be used during deployment instead of deploying specific implementation like pods or replicasets.

[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) creates [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) from manifest. Replicaset then maintains a set of identical pods.

[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) is similar to Deployment, except that it manages a replicaset of pods that are stateful, i.e. not interchangeable, the identity matters. But it has [limitations](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#limitations).

[Service](https://kubernetes.io/docs/concepts/services-networking/service/) is used to give pods network access. Each pod has its own IP but a set of pods has the same DNS name. Service is an abstraction layer to expose the app running on pods.

A [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) ensure there is a copy of a pod running on all/some of the nodes. Good for log collection, monitoring all the nodes, cluster storage, etc.
https://medium.com/stakater/k8s-deployments-vs-statefulsets-vs-daemonsets-60582f0c62d4

A [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) is a task runs only once, and [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) runs a task periodically, differentiate from deployment which is running incessantly.

Kubernetes supports [rolling updates](https://v1-19.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment). The Deployment updates Pods in a rolling update fashion when `.spec.strategy.type==RollingUpdate`. You can specify `maxUnavailable` and `maxSurge` to [proportionally control](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#proportional-scaling) the rolling update process. [Pausing/Resuming a deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment) `kubetctl rollout pause/resume deployment/${deployment-name}` is supported too.

#### rolling deployment commands
```
# check rollout status
kubectl rollout status deployment/${deployment-name}
# check the creation and exit of replicaset
kubectl get rs
# check pods status
kubectl get pods
# check if deployment running as expected
kubectl get deployment ${deployment-name}
# check description of current deployment
kubectl describe deployment ${deployment-name}

# check the rollout history of a deployment
kubectl rollout history deployment/${deployment-name}
# check the details of a rollout revision
kubectl rollout history deployment/${deployment-name} --revision=${revision-number}
# rollback to previous deployment
kubectl rollout undo deployment/${deployment-name}
# rollback to a specific revision
kubectl rollout undo deployment/${deployment-name} --to-revision=${revision-number}

# deployment status from command exit status
kubectl rollout
echo $? # 0 indicates success while 1 indicates error
```

#### deployment scale
```
# scale up/down a deployment
kubectl scale deployment/${deployment-name} --replicas=${replica-amount}
# horizontal pod autoscale: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
kubectl autoscale deployment/${deployment-name} --min=${min-amount-pods} --max=${max-amount-pods} --cpu-percent=${cpu-consumed-percentage, e.g. 80} 
#check the current status of the newly-made HorizontalPodAutoscaler
kubectl get hpa
```


## kubernetes Logging
- [basic logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/#basic-logging-in-kubernetes)
-- pods send data to standard output streams
-- `kubectl logs` to retrieve the logs
- [central cluster logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/#cluster-level-logging-architectures)
-- aggregate logs from all pods and stream it to central location
-- k8s has no default solution
-- ELK, EFL, etc

#### kubectl logs flags
```
kubectl logs --tail=30 ${pod-name} -n ${namespace} # last 30 logs
kubectl logs --since=5m ${pod-name} -n ${namespace} # logs from last 5m
kubectl logs -f ${pod-name} -n ${namespace} # stream the logs
kubectl logs -c ${container-name} ${pod-name} -n ${namespace} # specify container when pod got multiple containers
```

#### check logs on the node
```
# ssh into the node
kubectl get nodes -o wide # get node ip if not known
ssh ${user}@${node-ip}
# all the kube logs are under /var/log/pods
cd /var/log/pods
# every pod's log is within the dir named by its UID, describe the pod to see the UID
# if log file missing, check node for file system error, no free space error, etc
```

## Kubernetes Monitoring & Alerting
https://github.com/prometheus-operator/kube-prometheus for `kubeadm`
http://blog.itaysk.com/2019/01/15/Kubernetes-metrics-and-monitoring
https://www.networkcomputing.com/cloud-infrastructure/kubernetes-monitoring-5-key-metrics
https://www.brendangregg.com/usemethod.html for infra level
https://thenewstack.io/monitoring-microservices-red-method/ for app level


## Eviction
https://kubernetes.io/docs/concepts/scheduling-eviction/
https://devtron.ai/blog/ultimate-guide-of-pod-eviction-on-kubernetes/
