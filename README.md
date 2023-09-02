# Timescale Operator for Kubernetes

## What is this?

This is a helm chart for installing [Timescale Operator](https://github.com/hyperplane-org/timescale-operator)

## Installation

Add chart repo to Helm with:

```bash
helm repo add timescale-operator https://hyperplane-org.github.io/timescale-operator-helm/
```

Install to your cluster:

```bash
helm install timescale-operator timescale-operator/timescale-operator --namespace timescale-operator --create-namespace
```

## Deploy a Timescale cluster

At the time only [timescale/timescaledb-single](https://docs.timescale.com/self-hosted/latest/install/installation-kubernetes/) and [timescale/timescaledb-multinode](https://docs.timescale.com/self-hosted/latest/multinode-timescaledb/) can be deployed.
To deploy use the `TimescaledbMultinode` CRD installed by the operator:

```yaml
apiVersion: timescale.hyperplane.hu/v1alpha1
kind: TimescaledbSingle
metadata:
  name: mycluster
spec:
  image:
    pullPolicy: IfNotPresent
    repository: timescaledev/timescaledb-ha
    tag: pg14.6-ts2.9.1-p1
```

To deploy use the `TimescaledbMultinode` CRD installed by the operator:

```yaml
apiVersion: timescale.hyperplane.hu/v1alpha1
kind: TimescaledbMultinode
metadata:
  name: mycluster
spec:
  image:
    pullPolicy: IfNotPresent
    repository: timescaledev/timescaledb-ha
    tag: pg12-ts2.0.0-p0
```

## The full spec of the CRD is passed to the timescaledb/timescaledb-multinode Helm chart as values

The default values were copied from [timescale/timescaledb-multinode](https://docs.timescale.com/self-hosted/latest/multinode-timescaledb/)

```yaml
apiVersion: timescale.hyperplane.hu/v1alpha1
kind: TimescaledbMultinode
metadata:
  name: timescaledbmultinode-sample
spec:
  # The default values were copied from the timescaledb-multinode chart from here https://docs.timescale.com/self-hosted/latest/multinode-timescaledb/
  affinity: {}
  affinityTemplate: |
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: "kubernetes.io/hostname"
          labelSelector:
            matchLabels:
              app:  {{ template "timescaledb.name" . }}
              release: {{ .Release.Name | quote }}
  credentials:
    accessNode:
      superuser: tea
    dataNode:
      superuser: coffee
  dataNodes: 3
  env:
  - name: LC_ALL
    value: C.UTF-8
  - name: LANG
    value: C.UTF-8
  - name: PGDATA
    value: /var/lib/postgresql/pgdata
  image:
    pullPolicy: IfNotPresent
    repository: timescaledev/timescaledb-ha
    tag: pg12-ts2.0.0-p0
  nameOverride: timescaledb
  nodeSelector: {}
  persistentVolume:
    accessModes:
    - ReadWriteOnce
    annotations: {}
    enabled: true
    mountPath: /var/lib/postgresql
    size: 5G
    subPath: ""
  postgresql:
    databases:
    - postgres
    - example
    parameters:
      log_checkpoints: "on"
      log_connections: "on"
      log_line_prefix: '%t [%p]: [%c-%l] %u@%d,app=%a [%e] '
      log_lock_waits: "on"
      log_min_duration_statement: 1s
      log_statement: ddl
      max_connections: 100
      max_prepared_transactions: 150
      max_wal_size: 512MB
      min_wal_size: 256MB
      shared_buffers: 300MB
      temp_file_limit: 1GB
      timescaledb.passfile: ../.pgpass
      work_mem: 16MB
  rbac:
    create: true
  resources: {}
  serviceAccount:
    create: true
    name: null
  tolerations: []
```

## How?
[Timescale Operator](https://github.com/hyperplane-org/timescale-operator) is generated from [timescale/timescaledb-multinode](https://docs.timescale.com/self-hosted/latest/multinode-timescaledb/) with [operator-sdk](https://sdk.operatorframework.io/docs/building-operators/helm/quickstart/) with the Helm plugin.

## Why?
1. We deploy Timescale a lot
2. We wanted to be able to deploy Timescale from a CRD for an easier declarative config for our ArgoCD setup
3. [Timescale is not planning on doing this](https://github.com/timescale/timescaledb/issues/2982)

## Is this safe to use for production?

Probably not.
