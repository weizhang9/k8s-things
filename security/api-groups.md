kube's API has following groups:
![](../graph/kube-api-groups.png)

API: for core resources
![](../graph/kube-api-core.png)

APIS: for more detailed resources
![](../graph/kube-apis-named.png)

When curl API, need to specify tls config, unless use kubectl proxy
![](../graph/curl-api-tls.png)

When use kubectl proxy, kube proxy will use the kube config to query it for you.
![](../graph/curl-api-with-kube-proxy.png)

kube proxy != kubectl proxy
kube proxy is used to enable activities between different pods and services across nodes.
kubectl proxy is a http service created by kubtctl to access kube-apiserver.
