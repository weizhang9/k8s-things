## Kubernetes on AWS

### Example Setup
The architecture example
![Example AWS architecture](https://static.packt-cdn.com/products/9781788390071/graphics/assets/1043a804-7d8a-4850-a577-1c74900d0ccf.png "Example AWS architecture")
- assuming no existing aws infrastructure is set up
- two EC2 instances: one will run all the components for k8s control plane and worker nodes that runs your app; one as bastion host will allow incoming ssh traffic and connection to k8s clusters from private network

Step guide: https://www.golinuxcloud.com/setup-kubernetes-cluster-on-aws-ec2/

### Related Guides
- deploy EKS with terraform: https://www.golinuxcloud.com/terraform-eks-example/
- aws official doc for deploying k8s app: https://aws.amazon.com/getting-started/hands-on/deploy-kubernetes-app-amazon-eks/
- manual aws k8s deployment with `kubeadm`: https://piyushkashyap.net/2021/08/31/Self-managed-highly-available-kubernetes-cluster-on-AWS-using-kubeadm/

### [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)

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

#### k8s networking summary
- each pod is assigned its own IP address
- each pod can communicate with any other pod in the cluster without NAT
- the internal network that the software running inside a pod sees is identical to the pod network seen by other pods in the cluster

#### set up pod ip
AWS has an [CNI plugin](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html) for k8sto assign IP from your vpc to each pod.
```
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/v1.9/aws-k8s-cni.yaml
```