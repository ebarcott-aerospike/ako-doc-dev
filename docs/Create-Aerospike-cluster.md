---
title: Create Aerospike Cluster
description: Create Aerospike Cluster
---

To deploy the Aerospike cluster using the Operator, you will create an Aerospike custom resource file, which describes the cluster (including its number of nodes, the Aerospike configuration, system resources, etc.). You will then apply that configuration file to your Kubernetes cluster(s).

## Prerequisites

Before deploying your Aerospike cluster, you must [install the Aerospike Kubernetes Operator](Install-the-Operator-on-Kubernetes.md) on your Kubernetes cluster(s).

In order to follow along with this guide, download a copy of the Aerospike Kubernetes Operator GitHub repo (`aerospike-kubernetes-operator`) to your local computer. The repo contains scripts and other files which are used throughout this documentation as examples.

Either clone this repo or download the ZIP file to your computer [from the GitHub page](https://github.com/aerospike/aerospike-kubernetes-operator)

## Prepare the Aerospike Cluster Configuration

The Aerospike Kubernetes Operator GitHub repo contains example YAML configuration files for the cluster deployment. These files are located in the `aerospike-kubernetes-operator/config/samples` directory.

:::note
The following commands include file paths which are relative to the `aerospike-kubernetes-operator` directory. Be sure to either cd into this directory before you run the commands, or edit the command to include the correct file path for your system.
:::

The use case for your cluster will help you to determine which configuration parameters you need to set in the custom resource (CR) file. Identify your requirements for storage, if you plan to [enable XDR](XDR.md), or [manage TLS certificates](Manage-TLS-Certificates.md) for network security with your Aerospike clusters.

## Configure Persistent Storage

The Aerospike Operator is designed to work with dynamically-provisioned storage classes. Aerospike Server pods may have different storage volumes associated with each service.

Persistent storage on the pods will use these storage class provisioners to provision storage.

Apply a sample storage class based on your Kubernetes environment:

* EKS: `kubectl apply -f config/samples/storage/eks_ssd_storage_class.yaml`
* GCE: `kubectl apply -f config/samples/storage/gce_ssd_storage_class.yaml`
* Microk8s: `kubectl apply -f config/samples/storage/microk8s_filesystem_storage_class.yaml`

See [Storage Provisioning](Storage-provisioning.md) for more details on configuring persistent storage.

## Create Secrets

Next, create Secrets to set up features like the license file (`features.conf`), Aerospike authentication, TLS, and the cluster admin password. See the [Manage TLS Certificates](Manage-TLS-Certificates.md) section for more details.

The example `config/samples/secrets` directory includes a collection of example TLS certificates, security credentials, and more. Apply these files as a Kubernetes Secret:

```sh
kubectl  -n aerospike create secret generic aerospike-secret --from-file=config/samples/secrets
```

Create a Secrets containing the password for Aerospike cluster admin:

```sh
kubectl  -n aerospike create secret generic auth-secret --from-literal=password='admin123'
```

## Create Aerospike Cluster Custom Resource (CR)

Refer to the [cluster configuration settings](Cluster-configuration-settings.md) for details on the Aerospike cluster custom resource (CR) file. Sample Aerospike cluster CR files for different configurations can be found in the `aerospike-kubernetes-operator/config/samples` directory.

You can edit this file at any time to make changes and manage the Aerospike cluster.


## Deploy the Aerospike Cluster

Use the custom resource YAML file you created to deploy an Aerospike cluster. If you don't have a custom resource file, you can choose one of the sample files in the `config/samples` directory.

For example, to use the `config/samples/dim_nostorage_cluster_cr.yaml` file:

```sh
kubectl apply -f config/samples/dim_nostorage_cluster_cr.yaml
```

:::note
Replace the file name with the custom resource YAML file for your cluster.
:::

## Verify Cluster Status

Use `kubectl get statefulset -n aerospike` to ensure the aerospike-kubernetes-operator creates the StatefulSets for the custom resource.

```sh
$ kubectl get statefulset -n aerospike
NAME      READY   AGE
aerocluster-0   2/2     24s
```

Use `kubectl get pods -n aerospike` to check the pods to confirm the status. This step may take time as the pods provision resources, initialize, and are ready. Please wait for the pods to switch to the Running state before you continue.

```sh
$ kubectl get pods -n aerospike
NAME          READY   STATUS      RESTARTS   AGE
aerocluster-0-0     1/1     Running     0          48s
aerocluster-0-1     1/1     Running     0          48s
```

If the Aerospike cluster pods do not switch to Running status in a few minutes, please refer to the [Troubleshooting Guide](Troubleshooting.md).
