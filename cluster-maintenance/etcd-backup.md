`etcdctl` is a command line client for [etcd](https://github.com/coreos/etcd).

In some setup the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

Make sure you have the right `ETCDCTL_API` version. You can do this by exporting the variable `ETCDCTL_API` prior to using the etcdctl client.
```
export ETCDCTL_API=3
```

To see all the options for a specific sub-command, make use of the `-h` or `--help` flag.

For example, if you want to take a snapshot of etcd, use:

`etcdctl snapshot save -h` and keep a note of the mandatory global options.

If ETCD database is TLS-Enabled, the following options are mandatory:

1. --cacert: verify certificates of TLS-enabled secure servers using this CA bundle
2. --cert: identify secure client using this TLS certificate file
3. --endpoints=[127.0.0.1:2379]: this is the default as ETCD is running on master node and exposed on localhost 2379.
4. --key: identify secure client using this TLS key file

Similarly use the help option for `snapshot restore` to see all available options for restoring the backup.
```
etcdctl snapshot restore -h
```

Ref:
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

https://github.com/etcd-io/website/blob/main/content/en/docs/v3.5/op-guide/recovery.md

https://www.youtube.com/watch?v=qRPNuT080Hk
