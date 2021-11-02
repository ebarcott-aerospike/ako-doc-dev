---
title: HDD Storage With Data In Index
description: HDD Storage With Data In Index
---

Here we provide configuration for a specialized namespace where records have a single-bin and fit in 8 bytes.

For more details, visit [configuration of HDD Storage Engine with Data in Index Engine](https://docs.aerospike.com/docs/configure/namespace/storage/#recipe-for-a-hdd-storage-engine-with-data-in-index-engine).

## Create the namespace configuration
Following is the storage-specific config for the Aerospike cluster CR file.
```yaml
  storage:
    filesystemVolumePolicy:
      cascadeDelete: true
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
      - name: nstest
        aerospike:
          path: /opt/aerospike/data/test
        source:
          persistentVolume:
            storageClass: ssd
            volumeMode: Filesystem
            size: 3Gi
      - name: nsbar
        aerospike:
          path: /opt/aerospike/data/bar
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
    namespaces:
      - name: test
        memory-size: 2000000000
        single-bin: true
        data-in-index: true
        replication-factor: 1
        storage-engine:
          type: device
          files:
            - /opt/aerospike/data/test/test.dat
          filesize: 2000000000
          data-in-memory: true
      - name: bar
        memory-size: 3000000000
        single-bin: true
        data-in-index: true
        replication-factor: 1
        storage-engine:
          type: device
          files:
            - /opt/aerospike/data/bar/bar.dat
          filesize: 2000000000
          data-in-memory: true
```
Get full CR file [here](https://github.com/aerospike/aerospike-kubernetes-operator/tree/2.0.0-rc1/config/samples/hdd_dii_storage_cluster_cr.yaml).

## Deploy the cluster
Follow the instructions [here](Create-Aerospike-cluster.md#deploy-aerospike-cluster) to deploy this configuration.
