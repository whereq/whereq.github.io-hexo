---
title: Spark in Kubernetes
date: 2024-10-28 11:55:22
categories:
- Kubernetes
- Spark
tags:
- Kubernetes
- Spark
---

- [Architecture Overview](#architecture-overview)
  - [Diagram: Architecture Overview](#diagram-architecture-overview)
- [Prerequisites](#prerequisites)
- [1. Configuring Google Cloud Storage (GCS) Bucket](#1-configuring-google-cloud-storage-gcs-bucket)
- [2. Deploying PostgreSQL for Hive Metadata](#2-deploying-postgresql-for-hive-metadata)
  - [PostgreSQL YAML](#postgresql-yaml)
- [3. Installing Hive Metastore](#3-installing-hive-metastore)
  - [Hive Configuration](#hive-configuration)
- [4. Deploying Spark Master and Workers](#4-deploying-spark-master-and-workers)
  - [Spark Master and Worker StatefulSet YAML](#spark-master-and-worker-statefulset-yaml)
- [5. Setting Up the Spark History Server](#5-setting-up-the-spark-history-server)
  - [Spark History Server YAML](#spark-history-server-yaml)
- [6. Configuring Ingress](#6-configuring-ingress)
  - [Ingress YAML](#ingress-yaml)
- [7. Testing the Deployment with Spark Job](#7-testing-the-deployment-with-spark-job)
- [Furthermore](#furthermore)
  - [StatefulSet vs Deployment](#statefulset-vs-deployment)
  - [1. **Spark Master and Worker as StatefulSets**](#1-spark-master-and-worker-as-statefulsets)
  - [2. **Spark History Server as a Deployment**](#2-spark-history-server-as-a-deployment)
- [Summary Table](#summary-table)

---

This guide covers the steps to deploy an Apache Spark cluster with a master, two workers, and a Spark History Server on Kubernetes. It configures a GCS bucket as the default file system and installs Hive with PostgreSQL as its backend. An Ingress is set up for accessing the Spark UI and History Server.

<a name="architecture-overview"></a>
## Architecture Overview

The setup includes the following components:

- **Spark Master and Workers**: Deployed using StatefulSets for stable identity and persistence.
- **History Server**: Deployed using a Deployment and configured to store Spark event logs in GCS.
- **Hive and PostgreSQL**: PostgreSQL is used as the backend for Hive metadata, and Hive is deployed to store Spark tables.
- **Ingress**: Provides external access to Spark Master UI and History Server UI.

### Diagram: Architecture Overview

```
                +---------------------------+
                |                           |
                |        Kubernetes         |
                |                           |
                +---------------------------+
                |                           |
                |  +---------+   +---------+|
                |  |  Spark  |   |  Spark  ||
                |  |  Master |   | Workers ||
                |  +---------+   +---------+|
                |                           |
                |        +-----------+      |
                |        |  History  |      |
                |        |  Server   |      |
                |        +-----------+      |
                |            |              |
                |         Ingress           |
                +---------------------------+
                           |
                    External Access
```

---

<a name="prerequisites"></a>
## Prerequisites

- **Kubernetes Cluster** with access to deploy resources.
- **kubectl** command-line tool configured to access the cluster.
- **Helm** for deploying Hive and PostgreSQL.
- **Google Cloud Storage (GCS)** bucket created for Spark event logs.
- **Google Cloud SDK** installed and configured for authentication to GCS.

---

<a name="1-configuring-google-cloud-storage-gcs-bucket"></a>
## 1. Configuring Google Cloud Storage (GCS) Bucket

1. Create a GCS bucket if not already created:
   ```bash
   gsutil mb gs://your-spark-event-logs
   ```

2. Set up a GCS service account key and save it as a Kubernetes secret. This will allow Spark to access GCS:
   ```bash
   kubectl create secret generic gcs-key \
   --from-file=key.json=/path/to/gcs-service-account-key.json
   ```

---

<a name="2-deploying-postgresql-for-hive-metadata"></a>
## 2. Deploying PostgreSQL for Hive Metadata

PostgreSQL will act as the relational database for Hive’s metadata.

### PostgreSQL YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgresql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql
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
        env:
        - name: POSTGRES_DB
          value: "hive_metastore"
        - name: POSTGRES_USER
          value: "hiveuser"
        - name: POSTGRES_PASSWORD
          value: "hivepassword"
        volumeMounts:
        - mountPath: "/var/lib/postgresql/data"
          name: postgresql-data
      volumes:
      - name: postgresql-data
        persistentVolumeClaim:
          claimName: postgresql-pvc
```

---

<a name="3-installing-hive-metastore"></a>
## 3. Installing Hive Metastore

Hive Metastore will manage metadata for Spark. 

### Hive Configuration

1. Download and install Hive Helm chart:
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm install hive bitnami/hive --set postgresql.enabled=false
   ```

2. Update Hive configuration to use PostgreSQL.

```yaml
# Values.yaml for Hive
hive:
  metastore:
    databaseType: postgres
    jdbcUrl: jdbc:postgresql://postgresql-service/hive_metastore
    user: hiveuser
    password: hivepassword
```

---

<a name="4-deploying-spark-master-and-workers"></a>
## 4. Deploying Spark Master and Workers

### Spark Master and Worker StatefulSet YAML

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: spark-master
spec:
  serviceName: spark-master
  replicas: 1
  selector:
    matchLabels:
      app: spark
  template:
    metadata:
      labels:
        app: spark
        role: master
    spec:
      containers:
      - name: spark-master
        image: spark:latest
        ports:
        - containerPort: 7077
        - containerPort: 8080
        env:
        - name: SPARK_MASTER_HOST
          value: "spark-master"

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: spark-worker
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spark
      role: worker
  template:
    metadata:
      labels:
        app: spark
        role: worker
    spec:
      containers:
      - name: spark-worker
        image: spark:latest
        ports:
        - containerPort: 8081
        env:
        - name: SPARK_WORKER_CORES
          value: "1"
        - name: SPARK_WORKER_MEMORY
          value: "1G"
        - name: SPARK_MASTER
          value: "spark://spark-master:7077"
```

---

<a name="5-setting-up-the-spark-history-server"></a>
## 5. Setting Up the Spark History Server

The History Server will read event logs from the GCS bucket for job history.

### Spark History Server YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spark-history-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spark-history
  template:
    metadata:
      labels:
        app: spark-history
    spec:
      containers:
      - name: spark-history-server
        image: spark:latest
        env:
        - name: SPARK_HISTORY_OPTS
          value: "-Dspark.history.fs.logDirectory=gs://your-spark-event-logs"
        volumeMounts:
        - mountPath: /opt/spark/work-dir
          name: gcs-key
          subPath: key.json
      volumes:
      - name: gcs-key
        secret:
          secretName: gcs-key
```

---

<a name="6-configuring-ingress"></a>
## 6. Configuring Ingress

Ingress will expose the Spark Master UI and History Server UI.

### Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: spark-ingress
spec:
  rules:
  - host: spark.example.com
    http:
      paths:
      - path: /master
        pathType: Prefix
        backend:
          service:
            name: spark-master
            port:
              number: 8080
      - path: /history
        pathType: Prefix
        backend:
          service:
            name: spark-history-server
            port:
              number: 18080
```

---

<a name="7-testing-the-deployment-with-spark-job"></a>
## 7. Testing the Deployment with Spark Job

To verify the installation, use `spark-submit` to run a simple calculation for PI.

1. Submit a PI calculation job using `spark-submit`.

   ```bash
   kubectl exec -it spark-master-0 -- spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master spark://spark-master:7077 \
   /opt/spark/examples/jars/spark-examples_2.12-3.0.1.jar 1000
   ```

2. Check the Spark Master and History Server UIs

 to confirm job execution and view logs.


## Furthermore

### StatefulSet vs Deployment
In the official Helm chart for Apache Spark, the choice of **StatefulSet** for Spark Master and Worker, and **Deployment** for the Spark History Server, aligns with the specific requirements and behaviors of each component. Here’s a breakdown of the reasons for these design choices:

### 1. **Spark Master and Worker as StatefulSets**
   - **Persistence and Stability**: The Spark Master and Worker nodes need stable, unique identities to communicate reliably. Using **StatefulSet** ensures that each replica (instance) has a stable network identity (hostname) and, if configured, persistent storage.
   - **Leader Election and Failover**: Spark Master in standalone mode has a primary leader and backup node architecture, where the leader is responsible for the cluster state. StatefulSets allow these nodes to maintain consistent, unique identifiers, which is crucial for leader election and failover without disrupting the Spark cluster.
   - **Worker Registration**: Spark Workers register with the Master, and having a unique, stable network identity ensures workers don’t inadvertently re-register or conflict with each other after restarts or failures. StatefulSets are the preferred way to handle this kind of behavior.

### 2. **Spark History Server as a Deployment**
   - **Stateless Operation**: The Spark History Server is fundamentally a stateless application. It reads from a pre-configured storage (such as HDFS or S3) where application logs are stored but doesn’t require persistent storage for its own operation. Using a **Deployment** is appropriate here, as the history server can be easily replicated or scaled without maintaining unique identities.
   - **Easier Scaling and Update Management**: Deployments provide automatic scaling and rolling updates, which are more suitable for stateless services. If you need to increase the availability of the history server, you can simply increase the replica count in the Deployment, and Kubernetes will handle the scaling automatically.
   - **Failure Recovery**: Since there’s no dependency on unique network identities or persistent state, if the History Server pod fails, it can be easily recreated without any additional orchestration, which is exactly what Deployments are optimized for.

## Summary Table

| Component            | Kubernetes Kind | Purpose                                         | Reasoning                                                                                       |
|----------------------|-----------------|-------------------------------------------------|-------------------------------------------------------------------------------------------------|
| **Spark Master**     | StatefulSet     | Primary and backup master for Spark cluster     | Needs stable identity for cluster coordination and leader election                              |
| **Spark Worker**     | StatefulSet     | Worker nodes that register with Spark Master    | Stable network identity ensures reliable registration and communication with Master             |
| **Spark History Server** | Deployment     | Reads logs and provides historical application data | Stateless and doesn’t need unique identities; easier to scale and restart as a Deployment |

This setup optimizes the deployment structure for each Spark component's unique requirements, leveraging Kubernetes' StatefulSet and Deployment resources for their intended purposes.

