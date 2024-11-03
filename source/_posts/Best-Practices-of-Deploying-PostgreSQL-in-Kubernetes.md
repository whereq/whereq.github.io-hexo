---
title: Best Practices of Deploying PostgreSQL in Kubernetes
date: 2024-10-30 23:32:42
categories:
- Best Practices
- Kubernetes
- PostgreSQL
tags:
- Best Practices
- Kubernetes
- PostgreSQL
---

This guide walks through deploying PostgreSQL in a Kubernetes Pod with a sidecar container that collects PostgreSQL logs and forwards them to Elasticsearch. We’ll use Helm to manage deployments, including setting up necessary ConfigMaps and Secrets for configuration.

## Project Structure

```
postgresql-logging/
├── charts/
│   └── postgresql-log-sidecar/
│       ├── Chart.yaml
│       ├── templates/
│       │   ├── configmap.yaml
│       │   ├── deployment.yaml
│       │   ├── secret.yaml
│       │   ├── service.yaml
│       ├── values.yaml
├── README.md
```

## 1. Architecture Overview

The following diagram shows how the PostgreSQL Pod and the sidecar interact for log collection:

```
+-----------------------------------+
|        PostgreSQL Pod             |
|                                   |
|  +-----------------------------+  |
|  |  PostgreSQL Container       |  |
|  |  - Exposes Port 5432        |  |
|  +-----------------------------+  |
|                                   |
|  +-----------------------------+  |
|  |  Log Collector              |  |
|  |  - Collects Logs            |  |
|  |  - Pushes to Elasticsearch  |  |
|  +-----------------------------+  |
+-----------------------------------+
```

## 2. Helm Chart Configuration

### `Chart.yaml`

```yaml
apiVersion: v2
name: postgresql-log-sidecar
description: A Helm chart for PostgreSQL with log sidecar
version: 0.1.0
appVersion: "13"
```

### `values.yaml`

Define values for PostgreSQL configuration and the sidecar container.

```yaml
postgresql:
  username: admin
  password: password123
  database: appdb

elasticsearch:
  host: "elasticsearch:9200"
  index: "postgresql-logs"

sidecar:
  image: fluent/fluent-bit
  tag: "1.8"
  resources:
    limits:
      memory: "200Mi"
    requests:
      memory: "100Mi"
```

## 3. ConfigMap and Secret

### `configmap.yaml`

This ConfigMap includes log configuration for the sidecar container.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgresql-log-config
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush 1
        Log_Level info

    [INPUT]
        Name tail
        Path /var/log/postgresql/postgresql.log
        Tag postgresql

    [OUTPUT]
        Name  es
        Match *
        Host  {{ .Values.elasticsearch.host }}
        Port  9200
        Index {{ .Values.elasticsearch.index }}
```

### `secret.yaml`

Create a Secret for PostgreSQL credentials.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgresql-secret
type: Opaque
data:
  postgresql-username: {{ .Values.postgresql.username | b64enc }}
  postgresql-password: {{ .Values.postgresql.password | b64enc }}
```

## 4. Deployment Configuration

### `deployment.yaml`

The main deployment file sets up PostgreSQL and the Fluent Bit sidecar container.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql-log-sidecar
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      containers:
        - name: postgresql
          image: postgres:13
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_DB
              value: {{ .Values.postgresql.database }}
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgresql-secret
                  key: postgresql-username
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgresql-secret
                  key: postgresql-password
          volumeMounts:
            - name: postgres-logs
              mountPath: /var/log/postgresql
          resources:
            requests:
              memory: "500Mi"
            limits:
              memory: "1Gi"

        - name: log-collector
          image: {{ .Values.sidecar.image }}:{{ .Values.sidecar.tag }}
          args: ["-c", "/fluent-bit/etc/fluent-bit.conf"]
          volumeMounts:
            - name: postgres-logs
              mountPath: /var/log/postgresql
            - name: config
              mountPath: /fluent-bit/etc
              subPath: fluent-bit.conf
          resources:
            limits:
              memory: {{ .Values.sidecar.resources.limits.memory }}
            requests:
              memory: {{ .Values.sidecar.resources.requests.memory }}
      volumes:
        - name: postgres-logs
          emptyDir: {}
        - name: config
          configMap:
            name: postgresql-log-config
```

### `service.yaml`

Create a Service to expose PostgreSQL internally within the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgresql
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: postgresql
```

## 5. Deploying the Helm Chart

Install the chart with:

```bash
helm install postgresql-log-sidecar ./charts/postgresql-log-sidecar
```