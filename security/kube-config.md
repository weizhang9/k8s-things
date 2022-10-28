set kube config so that we don't need to pass in ca, key, cert files with every request.
![](../graph/kube-config.png)

current context shows the current context being used.
![](../graph/current-context.png)

use `k config use-context <user@context>` to switch current context.
![](../graph/kube-config-context.png)

kube config has ns field for namespace, has cert fields for certs
![](../graph/kube-config-ns-cert.png)
certs can be specified by path or base64 encoded content.
![](../graph/kube-config-cert-data.png)
