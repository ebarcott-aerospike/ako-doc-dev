---
title: Data On SSD
description: Data On SSD
---

Here we provide namespace storage configuration for storing namespace data on a provisioned SSD storage device.

For more details, visit [configuration of SSD Storage Engine](https://docs.aerospike.com/docs/configure/namespace/storage/#recipe-for-an-ssd-storage-engine).

## Create the namespace configuration
Following is the Storage specific config for aerospike cluster CR file.

```yaml
  storage:
    filesystemVolumePolicy:
      initMethod: deleteFiles
      cascadeDelete: true
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
      - name: ns
        aerospike:
          path: /test/dev/xvdf
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
            - /test/dev/xvdf
```
Get full CR file [here](https://github.com/aerospike/aerospike-kubernetes-operator/tree/2.0.0-rc1/config/samples/ssd_storage_cluster_cr.yaml).

## Deploy the cluster
Follow the instructions [here](Create-Aerospike-cluster.md#deploy-aerospike-cluster) to deploy this configuration.
