## Kubernetes on AWS

<!-- `make toc` to generate https://github.com/jonschlinkert/markdown-toc#cli -->

<!-- toc -->

- [Example Setup](#example-setup)
- [Related Guides](#related-guides)
- [kubeadm](#kubeadm)
  * [what does it do?](#what-does-it-do)
  * [what's inside `/etc/kubernetes/manifests`?](#whats-inside-etckubernetesmanifests)
- [Pod Networking](#pod-networking)
  * [k8s networking summary](#k8s-networking-summary)
  * [set up pod ip](#set-up-pod-ip)
- [Production Planning](#production-planning)
  * [design process](#design-process)
  * [design requirements](#design-requirements)

<!-- tocstop -->

### Example Setup
---
The architecture example  
<img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/1043a804-7d8a-4850-a577-1c74900d0ccf.png" alt="Example AWS architecture" width="80%" />

- assuming no existing aws infrastructure is set up
- two EC2 instances: one will run all the components for k8s control plane and worker nodes that runs your app; one as bastion host will allow incoming ssh traffic and connection to k8s clusters from private network

Step guide: https://www.golinuxcloud.com/setup-kubernetes-cluster-on-aws-ec2/

### Related Guides
---
- [example](https://www.golinuxcloud.com/terraform-eks-example/) of deploying EKS with terraform
- aws official doc for [deploying k8s app](https://aws.amazon.com/getting-started/hands-on/deploy-kubernetes-app-amazon-eks/)
- [example](https://github.com/PacktPublishing/Kubernetes-on-AWS/tree/master/chapter07) of using terraform to prepare node images and a node group and provision add-ons
- manual aws k8s deployment with `kubeadm`: https://piyushkashyap.net/2021/08/31/Self-managed-highly-available-kubernetes-cluster-on-AWS-using-kubeadm/

### [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)
---
#### what does it do?

1. establish a private key infrastructure
2. write static pod manifests to the `/etc/kubernetes/manifests` directory
3. `kubelet` is then configured to read these static pod manifests in a `systemd dropin` that `kubeadm` creates at `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
4. once the API server's started, `kubeadm` submits 2 add-ons to the API:  
   -- `kube-proxy`: the process that configures iptables on each node to make the service IPs route correctly, it's run on each node with a DaemonSet.  
   -- `kube-dns`: the process that provides the DNS server, can be used by apps running on the cluster for service discovery, **it won't run correctly until a pod network for the cluster is configured**.

#### what's inside `/etc/kubernetes/manifests`?

- `etcd.yaml`: the key-value store of k8s API server's state
- `kube-apiserver.yaml`: API server config
- `kube-controller-manager.yaml`: control manager config
- `kube-scheduler.yaml`: scheduler config

> pro tip: add `.kube/config` inside your aws node to your local machine's `.kube/config`, you can access aws pods without ssh into aws node 😏 (ok for test env but vuln consideration for prod)

### Pod Networking
---
#### k8s networking summary

- each pod is assigned its own IP address
- each pod can communicate with any other pod in the cluster without NAT
- the internal network that the software running inside a pod sees is identical to the pod network seen by other pods in the cluster

#### set up pod ip

AWS has an [CNI plugin](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html) for k8sto assign IP from your vpc to each pod.

```
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/v1.9/aws-k8s-cni.yaml
```

### Production Planning
---
#### design process

- avoid changing too much too quickly, avoid scope creep and try to change as little as possible to deliver your intial adoption of k8s as quickly as possible.
- consider the current state of your app deployment, and aim to replicate its facilities first, additional functionalities can be added later
- if you have the opportunity to reduce the scope of the infrastructure your k8s deployment provides at the time of your additional roll out, you should consider doing so
- if you currently deploy your app onto servers running in a linux distro that is readily available as a container image, you should stick with that distro rather than looking for alternatives at this stage, as the team would already be knowledgeable about it, so that you won't have to invest time fixing imcompatibilities
- you should wait to successfully establish the use and operation of k8s first, as a stable foundation will put you in a much better position to introduce new tools and processes
- you should view the adoption of k8s as building a foundation that will be flexible enough to implement whatever changes to tools and processes you want to make in the future

#### design requirements

<img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/5a21d4ce-edfe-4d5e-9fa1-f901d1f353fc.png" alt="design requirements diagram" width="50%" />

- availability  
  <img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/ace2923e-7d64-418e-90af-66ec4b2d60bd.png" alt="availablity diagram" width="50%" />

- capacity  
  <img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/c53ff269-a214-41be-9afa-eeec3728a8a6.png" alt="capacity diagram" width="50%" />  
  
  -- **EC2 instance type**  
  - When preparing a cluster, we should think about the instance types and size of instances that make up our cluster
  - The CPU/Memory ratio our pods request should be inline with ratio in EC2 instance type as close as possible to not waste resources

  <table style="border-collapse: collapse;width: 100%" border="3">
  <tbody>
    <tr>
      <td><strong>Category</strong></td>
      <td><strong>Type</strong></td>
      <td><strong>CPU to memory ratio: vCPU:GiB</strong></td>
      <td><strong>Notes</strong></td>
    </tr>
    <tr>
      <td>Burstable</td>
      <td>T3</td>
      <td>1 CPU : 2 GiB</td>
      <td>Provides 5-40% CPU baseline + burstable extra use.</td>
    </tr>
    <tr>
      <td>CPU optimized</td>
      <td>C5</td>
      <td>1 CPU : 2 GiB</td>
      <td></td>
    </tr>
    <tr>
      <td>General purpose</td>
      <td>M5</td>
      <td>1 CPU : 4 GiB</td>
      <td></td>
    </tr>
    <tr>
      <td>Memory optimized</td>
      <td>R5</td>
      <td>1 CPU : 8 GiB</td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td>X1</td>
      <td>1 CPU : 15GiB</td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td>X1e</td>
      <td>1 CPU : 30GiB</td>
      <td></td>
    </tr>
    <tr>
      <td colspan="4">You should only consider the following instance types if you need the additional extra resources they provide (GPUs and/or local storage):</td>
    </tr>
    <tr>
      <td>GPU</td>
      <td>P3</td>
      <td>1 CPU : 7.6GiB</td>
      <td>1 GPU : 8 CPU (NVIDIA Tesla V100)</td>
    </tr>
    <tr>
      <td></td>
      <td>P2</td>
      <td>1 CPU : 4GiB</td>
      <td>i. 1 GPU : 4 CPU (NVIDIA K80)</td>
    </tr>
    <tr>
      <td>Storage</td>
      <td>H1</td>
      <td>1 CPU : 4GiB</td>
      <td>2TB HDD : 8 CPU</td>
    </tr>
    <tr>
      <td></td>
      <td>D2</td>
      <td>1 CPU : 7.6GiB</td>
      <td>3TB HDD : 2 CPU</td>
    </tr>
    <tr>
      <td></td>
      <td>I3</td>
      <td>1 CPU : 7.6GiB</td>
      <td>475GiB SSD : 2 CPU</td>
    </tr>
  </tbody>
  </table>
  
  -- **breadth vs depth**
  - the size of your instances limits the largest pod you can run on your cluster. the instance needs 10-20% larger than your largest pod to account for the overhead of system services, such as logging or monitoring tools, docker, and kubernetes itself
  - smaller instances will allow you to scale your cluster in smaller increments, increasing utilization
  - fewer (larger) instances may be simpler to manage
  - larger instances may use a lower proportion of resources for cluster-level tasks, such as log shipping, and metrics
  - if you want to use monitoring or logging tools, such as datadog, sysdig, newrelic, and so on, where pricing is based on a per instance model, fewer larger instances may be more cost effective
  - larger instances can provide more disk and networking bandwidth, but if you are running more processes per instance this may not offer any advantage
  - larger instance sizes are less likely to suffer from noisy neighbor issues at the hypervisor level
  - larger instances often imply more colocation of your pods. this is usually advantageous when the aim is to increase utilization, but can sometimes cause unexpected patterns of resource limitations
  
- performance  
  <img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/7d5d9899-71d1-4f75-b644-c548bd23330a.png" alt="key components impacting performace" width="50%" /> 
  
  -- **disk performance**: if you app depends on disk performance, understanding EBS volumes attached to your EC2 instance is important.
  - all of the current generation of EC2 instances relies on EBS storage. EBS storage is effectively a shared network attached storage
  - EBS provides four volume types: `gp2` and `io2` volumes are based on SSD while `st1` and `sc1` volumes are based upon HDD
  - hard limit: the maximum IOPS available to all EBS volumes attached to a single instance is 64,000 on the largest instance size available on c5 and m5 instances, the smallest c5 and m5 instances only provide 1,600 IOPS

  -- **network performance**: on aws linux 2 machine, ena are auto-loaded, but only the newer generation of instance type is using the ena driver. to improve network performance:
  - increasing instance size: this makes faster networking available to the instance, and also increases collocation so making localhost network calls between services more likely
  - adding your instance to a cluster placement group: this ensures that your instances are hosted physically nearby, improving network performance

- security  
  <img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/989337d6-4468-42f6-9a96-069308556c62.png" alt="key areas impacting performace" width="70%" />  
  -- **patch cycle**: k8s community currently supports three minor versions at any one time, ending regular patch-level updates of the oldest supported version as each new minor version is released
  - patch-level updates: up to several times a month, trivial effort, very little to no downtime
  - minor-version upgrades: every 3-9 months, may need to make minor config changes, k8s maintains good backwards compatibility, deprecation will show in warning output, dependencies using beta/alpha apis might need to be updated first
  - have a test env is critical, and have a strategy/tool/procedure to rollback to any previous upgrade, and a good monitoring

  -- **network security**: when running k8s on aws, tehre are 4 layers to configure in order to correctly secure the traffic on your cluster:
  - *infra-node networking*: for traffic to pass thru, you will need to configure aws SGs applied to your nodes.  
  when using an overlay network, it typically means allowing traffic on a particular port as all communication is encapsulated to pass over a single port.  
  when using a native vpc networking like `amazon-vpc-cni-k8s`, it is typically necessary to allow all traffic to pass between the nodes, as the plugin associates multiple pod IPs with a single ENI, so it is not typically possible to manage infra-node networking in a more granular way using SGs.

  - *node-master networking*: `kubelet` running on the nodes normally needs to connect to k8s api to discover the definitions of the pods it's expected to be running.  
  typically this means allowing k8s worker nodes to make tcp connections on port `443` to control plane's SG.  
  the control plane connects to the `kubelet` on an api exposed on port `10250`, this is needed for the `logs` and `exec` functionalities.

  - *external networking*: think carefully about the services exposed to the wider internet, e.g.  k8s dashboard has access to the cluster, limit this to specific IPs.  
  when exposing a service/ingress controller to the wider internet, the [`LoadBalancer` service](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) type will configure appropriate SGs and provisioning an ELB for you

  - *k8s infra-pod networking*: k8s allows any pod running on the same cluster to connect with any other pod or service.  
  if you want to provide policies to restrict it between different apps running on your cluster, you will need to deploy a network plugin that enforce k8s networking policies, like Calico, Romana or WeaveNet. Calico is preferred as it's [AWS-supported](https://github.com/aws/amazon-vpc-cni-k8s/blob/master/config/master/calico-operator.yaml).  
  k8s `NetworkPolicy` targets at pods thru label selector and namespace. you can set a default `NetworkPolicy` that blocks traffic for pods that haven't yet set a specific network policy.

  -- **IAM role**: k8s ships with good aws integration, it can provision EBS volumes and attach them to the EC2 instances in your cluster, set up ELB, configure SGs, etc on your behalf. you will need to set up IAM roles to allow the control plane and nodes to perform these tasks.  
  - the easiest way to do so is to attach an instance profile associated with a relevant IAM role to grant k8s processes running on the instance that required permissions.  
  - not sharing IAM roles among different environments is good practice.  
  - most resources k8s interacts with are tagged with a `kubernetes.io/cluster/<cluster name>` tag, which can be used to restrict certain actions, like deletion.
  - [aws supports associating IAM role with a k8s service account](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). tools like [kube2iam](https://github.com/jtblin/kube2iam) and [kiam](https://github.com/uswitch/kiam) are also available but not usually needed.  
  
  -- **validation**: set up k8s conformance tests against your cluster to make sure it's operating correctly and before any code change is merged.
  - [sonobuoy](https://github.com/heptio/sonobuoy) helps to run k9s conformance tests on your clusters
  - you can also follow the k8s security benchmark checklist on https://www.cisecurity.org/benchmark/kubernetes/, which can be useful to follow when building the cluster
  - [kube-bench](https://github.com/aquasecurity/kube-bench) helps automating the k8s security benchmark against your cluster

- observability  
  <img src="https://static.packt-cdn.com/products/9781788390071/graphics/assets/e1f09642-520c-4579-809e-3ac616965ea2.png" alt="observability diagram" width="70%" /> 

  -- **logging**: `kubectl logs` out of box, use external tool for log manipulation

  -- **monitoring**: better use a tool k8s already intergates with, keep it simple, e.g.
  - expose an endpoint `/metrics` on port `9102`
  - add the annotation `"prometheus.io/scrape": true` to your pod

  -- **alerting**: should be actionable, directed and used sparingly  

  -- **tracing**: particularly useful for microservices, tools like [aws xray](https://aws.amazon.com/xray/) and [opentelemetry](https://opentelemetry.io/)
