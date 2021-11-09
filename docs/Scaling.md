---
title: Scaling
description: Scaling
---

<!-- CONTENT AUDIT: This page seems unnecessary. Maybe it can go in a separate "examples" section. -->

For this example assume that cluster is deployed using a file named `aerospike-cluster.yaml`.

Change the `spec.size` field in the yaml file to scale the cluster up or down to the specified number of pods.

```yaml
apiVersion: asdb.aerospike.com/v1beta1
kind: AerospikeCluster
metadata:
  name: aerocluster
  namespace: aerospike
spec:
  size: 2
  image: aerospike/aerospike-server-enterprise:4.9.0.33
  .
  .
```

Use kubectl to apply the change.

```sh
$ kubectl apply -f aerospike-cluster.yaml
```

Check the pods.

```sh
$ kubectl get pods -n aerospike
NAME          READY   STATUS      RESTARTS   AGE
aerocluster-0-0     1/1     Running     0          3m6s
aerocluster-0-1     1/1     Running     0          3m6s
aerocluster-0-2     1/1     Running     0          30s
aerocluster-0-3     1/1     Running     0          30s
```
