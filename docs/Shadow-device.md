---
title: Shadow Device
description: Shadow Device
---

## Description
This is specific to cloud environments. Here namespace storage-engine can be configured to use extremely high-performance cloud instance attached local SSDs that are ephemeral. Writes will also be duplicated to another network-attached shadow device for persistence in case the cloud instance terminates.

For more details, visit [configuration of shadow devices](https://docs.aerospike.com/docs/configure/namespace/storage/#recipe-for-shadow-device).


## Create a local provisioner and local storage class
Follow the instructions [here](Storage-provisioning.md#local-volume) to create a local volume provisioner and appropriate storage class.

## Create the namespace configuration
Storage specific config for aerospike cluster CR file.
```yaml
  storage:
    filesystemVolumePolicy:
      cascadeDelete: true
      initMethod: deleteFiles
    blockVolumePolicy:
      cascadeDelete: true
    volumes:
      - name: workdir
        aerospike:
          path: /opt/aerospike
        source:
          persistentVolume:
            storageClass: ssd
            volumeMode: Filesystem
            size: 1Gi
      - name: nsvol1
        aerospike:
          path: /dev/nvme0n1
        source:
          persistentVolume:
            storageClass: local-ssd
            volumeMode: Block
            size: 5Gi
      - name: nsvol2
        aerospike:
          path: /dev/sdf
        source:
          persistentVolume:
            storageClass: ssd
            volumeMode: Block
            size: 5Gi
      - name: aerospike-config-secret
        source:
          secret:
            secretName: aerospike-secret
        aerospike:
          path: /etc/aerospike/secret
  .
  .
  .
  aerospikeConfig:
    service:
      feature-key-file: /etc/aerospike/secret/features.conf
    security:
      enable-security: true
    namespaces:
      - name: test
        memory-size: 3000000000
        replication-factor: 2
        storage-engine:
          type: device
          devices:
            - /dev/nvme0n1 /dev/sdf
```
Get full CR file [here](https://github.com/aerospike/aerospike-kubernetes-operator/tree/2.0.0-rc1/config/samples/shadow_device_cluster_cr.yaml).

## Deploy the cluster
Follow the instructions [here](Create-Aerospike-cluster.md#deploy-aerospike-cluster) to deploy this configuration.
