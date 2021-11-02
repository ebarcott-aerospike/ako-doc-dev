---
title: HDD Storage With Data In Memory
description: HDD Storage With Data In Memory
---

Here we provide namespace storage configuration for storing namespace data both in memory and on the persistent device as well.

For more details, visit [HDD Storage Engine with Data in Memory](https://docs.aerospike.com/docs/configure/namespace/storage/#recipe-for-an-hdd-storage-engine-with-data-in-memory)

## Create the namespace configuration
Following is the storage-specific config for the Aerospike cluster CR file.
```yaml
  storage:
    filesystemVolumePolicy:
      cascadeDelete: false
      initMethod: deleteFiles
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
          path: /opt/aerospike/data
        source:
          persistentVolume:
            storageClass: ssd
            volumeMode: Filesystem
            size: 3Gi
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
    namespace:
      - name: test
        memory-size: 3000000000
        replication-factor: 2
        storage-engine:
          type: device
          file:
            - /opt/aerospike/data/test.dat
          filesize: 2000000000
          data-in-memory: true
```
Get full CR file [here](https://github.com/aerospike/aerospike-kubernetes-operator/tree/2.0.0-rc1/config/samples/hdd_dim_storage_cluster_cr.yaml).

## Deploy the cluster
Follow the instructions [here](Create-Aerospike-cluster.md#deploy-aerospike-cluster) to deploy this configuration.
