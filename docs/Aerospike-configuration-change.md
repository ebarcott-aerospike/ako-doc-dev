---
title: Modify Aerospike cluster
description: Modify Aerospike cluster
---

<!-- CONTENT AUDIT: Should this page continue to exist? It's basically just saying "make your change and use kubectl to apply it." This should be common knowledge for our assumed user base. -->

For this example assume that cluster is deployed using a file named `aerospike-cluster.yaml`.

## Example Configuration Change

Change the `spec.aerospikeConfig.service.proto-fd-max` field in the aerocluster CR to `20000`

```yaml
apiVersion: asdb.aerospike.com/v1beta1
kind: AerospikeCluster
metadata:
  name: aerocluster
  namespace: aerospike
spec:
  size: 2
  image: aerospike/aerospike-server-enterprise:4.9.0.33
  aerospikeConfig:
    service:
      proto-fd-max: 15000
  .
  .
```

Use kubectl to apply the change.

```sh
$ kubectl apply -f aerospike-cluster.yaml
```

## Check the Pods

Pods will undergo a rolling restart.

```sh
$ kubectl get pods -n aerospike
NAME          READY   STATUS              RESTARTS   AGE
aerocluster-0-0     1/1     Running         0          3m6s
aerocluster-0-1     1/1     Running         0          3m6s
aerocluster-0-2     1/1     Running         0          30s
aerocluster-0-3     1/1     Terminating     0          30s
```
After all the pods have restarted, use `kubectl describe` to get status of the cluster.

Check `spec.aerospikeConfig.service.proto-fd-max` in status.

```sh
$ kubectl -n aerospike describe aerospikecluster aerocluster
Name:         aerocluster
Namespace:    aerospike
Kind:         AerospikeCluster
.
.
Status:
  Aerospike Config:
    Service:
      Proto - Fd - Max:   20000
  .
  .
```
