Taints and toleration guarantees a tainted node only accepts pods with matched toleration, but it doesn't guarantee the pods with matched toleration doesn't prefer a node without any taints.

Node affinity guarantees pods only gets scheduled onto nodes with matched requirements but it doesn't guarantee other pods without any affinity rules not get scehduled onto the same node.

As such, to guarantee a specific pod placed on a specific node, we need to combine taints and affinity together.
