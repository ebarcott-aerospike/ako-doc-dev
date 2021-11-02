---
title: Multiple Aerospike Clusters
description: Multiple Aerospike Clusters
---

The operator is able to deploy multiple Aerospike clusters within a single k8s namespace or in multiple k8s namespaces. 

## Multiple Aerospike clusters in a single k8s namespace

Deploying multiple clusters in a single namespace is as easy as deploying a single cluster. The user has to just deploy another cluster with a cluster name (cluster object metadata name in cr.yaml file) that is not already registered in that namespace.

## RBAC for other namespaces
For deploying and managing Aerospike cluster's in the operator's namespace (recommended as aerospike) you need not do any additional RBAC configuration.

For deploying cluster in namespaces other than the operator's namespace you need to
- create a service account with name `aerospike-operator-controller-manager` in that namespace
- add this service account to the operator's `ClusterRoleBinding`

#### Create the operator service account for the namespace

```shell
# Replace ns1 with your target namespace
kubectl -n ns1 create  serviceaccount aerospike-operator-controller-manager
```

#### Add service account to operator's ClusterRoleBinding
Find the ClusterRoleBinding created for the operator and add the service account created above

```shell
kubectl get clusterrolebindings.rbac.authorization.k8s.io  | grep aerospike-kubernetes-operator
aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d                 ClusterRole/aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d   41m
```

In the example above the name of the cluster role binding is `aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d`

Edit the role binding and add a new subject entry for the service account

```shell
# Replace aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d with the name of the cluster role binding found above
kubectl edit clusterrolebindings.rbac.authorization.k8s.io  aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d
```

In the editor that is launched append the following lines to the subjects section as shown below

```yaml
  # A new entry for ns1.
  # Replace ns1 with your namespace
  - kind: ServiceAccount
    name: aerospike-operator-controller-manager
    namespace: ns1
```

Save and ensure that the changes are applied.

Here is a full example of the operator's ClusterRoleBinding targeting `aerospike` and `ns1` namespaces.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2021-09-16T10:48:36Z"
  labels:
    olm.owner: aerospike-kubernetes-operator.v2.0.0-rc1
    olm.owner.kind: ClusterServiceVersion
    olm.owner.namespace: test
    operators.coreos.com/aerospike-kubernetes-operator.test: ""
  name: aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d
  resourceVersion: "51841234"
  uid: be546dd5-b21e-4cc3-8a07-e2fe5fe5274c
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: aerospike-kubernetes-operator.v2.0.0-rc1-74b946466d
subjects:
  - kind: ServiceAccount
    name: aerospike-operator-controller-manager
    namespace: aerospike

  # New entry
  - kind: ServiceAccount
    name: aerospike-operator-controller-manager
    namespace: ns1     
```
