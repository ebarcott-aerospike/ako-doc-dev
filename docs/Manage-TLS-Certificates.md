---
title: Manage TLS Certificates
description: Manage TLS Certificates
---

Here we describe setting up a TLS enabled Aerospike cluster.

For more details, visit [TLS configuration](https://docs.aerospike.com/docs/configure/network/tls/).

## Create a secret containing TLS certificates and key.

Assuming your TLS secrets are in config/samples/secrets folder, create a Kubernetes secret like so
```sh
$ kubectl create secret generic aerospike-secret --from-file=config/samples/secrets -n aerospike
```

## Create the TLS specific Aerospike configuration.
TLS specific config for the Aerospike cluster CR file.

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
      - name: aerospike-config-secret
        source:
          secret:
            secretName: aerospike-secret
        aerospike:
          path: /etc/aerospike/secret

  aerospikeConfig:
    service:
      feature-key-file: /etc/aerospike/secret/features.conf
    security:
      enable-security: true
    network:
      service:
        tls-name: aerospike-a-0.test-runner
        tls-authenticate-client: any
        tls-port: 4333
      heartbeat:
        tls-name: aerospike-a-0.test-runner
        tls-port: 3012
      fabric:
        tls-name: aerospike-a-0.test-runner
        tls-port: 3011
      tls:
        - name: aerospike-a-0.test-runner
          cert-file: /etc/aerospike/secret/svc_cluster_chain.pem
          key-file: /etc/aerospike/secret/svc_key.pem
          ca-file: /etc/aerospike/secret/cacert.pem

```
Get full CR file [here](https://github.com/aerospike/aerospike-kubernetes-operator/tree/2.0.0-rc1/config/samples/tls_cluster_cr.yaml).

## Deploy the cluster
Follow the instructions [here](Create-Aerospike-cluster.md#deploy-aerospike-cluster) to deploy this configuration.


