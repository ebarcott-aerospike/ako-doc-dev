---
title: Monitoring
description: Monitoring
---

The Aerospike Monitoring Stack is a useful way to enable monitoring and alerts for Aerospike clusters deployed by the Aerospike Kubernetes Operator.

## Add Aerospike Prometheus Exporter Sidecar

Add the exporter as a sidecar to each Aerospike server pod using the [PodSpec configuration](Cluster-configuration-settings.md#pod-spec).

```yaml
spec:
 .
 .
 .

  podSpec:
    sidecars:
     - name: aerospike-prometheus-exporter
       image: aerospike/aerospike-prometheus-exporter:1.3.0
       ports:
         - containerPort: 9145
           name: aerospike-prometheus-exporter

 .
 .
 .
```

Create or update your clusters after the Prometheus exporter sidecar is added.

## Prometheus Configuration

Configure Prometheus to add exporter endpoints as scrape targets.

If Prometheus is also running on Kubernetes, it can be configured to extract exporter targets from the Kubernetes API.

In the following example, Prometheus will be able to discover and add exporter targets in the `default` namespace which has endpoint port name of `aerospike-prometheus-exporter`.

```yaml
scrape_configs:
  - job_name: 'aerospike'

    kubernetes_sd_configs:
    - role: endpoints
      namespaces:
        names:
        - default
    relabel_configs:
    - source_labels: [__meta_kubernetes_endpoint_port_name]
      separator: ;
      regex: aerospike-prometheus-exporter
      replacement: $1
      action: keep
```

See the Aerospike Database documentation for more information on installing and configuring the Aerospike Monitoring Stack.

## Example

This example demonstrates how to use the Aerospike Monitoring Stack to monitor Aerospike clusters deployed by the Aerospike Kubernetes Operator.

Deploy the Aerospike Kubernetes Operator using OLM [as described in the Getting Started section](Create-Aerospike-cluster.md).

Create a Kubernetes Secret `aerospike-license` to store the Aerospike license feature key file.

```shell
kubectl create secret generic aerospike-license --from-file=[path to the features.conf-file]
```

Deploy an Aerospike cluster with an Aerospike Prometheus Exporter sidecar.

```shell
cat << EOF | helm install aerospike aerospike/aerospike-cluster \
--set devMode=true \
--set aerospikeSecretName=aerospike-license \
-f -
podSpec:
  sidecars:
  - name: aerospike-prometheus-exporter
    image: "aerospike/aerospike-prometheus-exporter:1.3.0"
    ports:
    - containerPort: 9145
      name: exporter
EOF
```

Deploy Prometheus-Grafana Stack using [aerospike-monitoring-stack.yaml](https://docs.aerospike.com/docscloud/assets/aerospike-monitoring-stack.yaml).

```shell
kubectl apply -f ./aerospike-monitoring-stack.yaml
```

Connect to the Grafana dashboard.

```shell
kubectl port-forward service/aerospike-monitoring-stack-grafana 3000:80
```

Open `localhost:3000` in browser, and login to Grafana as `admin`/`admin`.

Import dashboards from [Aerospike Monitoring GitHub Repo](https://github.com/aerospike/aerospike-monitoring/tree/master/config/grafana/dashboards) to view the metrics.
