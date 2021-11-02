---
title: XDR
description: XDR
---

To deploy a cluster as XDR source, you should configure `dc-security-config-file` config in CR file in `aerospikeConfig.xdr.datacenter` section. Also configure `dc-node-address-port` in same section for destination DC. After configuring these values in the CR file  apply CR file to deploy the cluster.

For more details, visit [configure cross-datacenter](https://docs.aerospike.com/docs/configure/cross-datacenter/)

## Enable XDR and create a remote DC
Following is the XDR specific config for the Aerospike cluster CR file.
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
    - name: ns
      aerospike:
        path: /opt/aerospike/data
      source:
        persistentVolume:
          storageClass: ssd
          volumeMode: Filesystem
          size: 3Gi
    - name: xdr
      aerospike:
        path: /opt/aerospike/xdr
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
        
  aerospikeConfig:
    logging:
      - name: /var/log/aerospike/aerospike.log
        any: info

    service:
      feature-key-file: /etc/aerospike/secret/features.conf

    security:
      enable-security: true

    network:
      service:
        port: 3000
      fabric:
        port: 3001
      heartbeat:
        port: 3002

    xdr:
      dcs:
        - name: dc1
          node-address-ports:
            - aeroclusterdst-0-0 3000

          auth-user: admin
          auth-password-file: /etc/aerospike/secret/password_DC1.txt
          namespaces:
            - name: test

    namespaces:
      - name: test
        memory-size: 3000000000
        replication-factor: 2
        storage-engine:
          type: device
          files:
            - /opt/aerospike/data/test.dat
          filesize: 2000000000
          data-in-memory: true

```
Get full CR file [here](https://github.com/aerospike/aerospike-kubernetes-operator/tree/2.0.0-rc1/config/samples/xdr_src_cluster_cr.yaml).

## Remote DC Credentials
If destination cluster is security enabled then `aerospike-secret` created in this section should also have `security_credentials_DC1.txt` file for destination DC.

```sh
$ cat security_credentials_DC1.txt
credentials
{
   username xdr_user
   password xdr_pass
}
```

## Deploy the cluster
Follow the instructions [here](Create-Aerospike-cluster.md#deploy-aerospike-cluster) to deploy this configuration.
