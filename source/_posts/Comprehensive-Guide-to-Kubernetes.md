---
title: Comprehensive Guide to Kubernetes
date: 2024-10-28 11:51:24
categories:
- Kubernetes
tags:
- Kubernetes
---

- [Cluster](#cluster)
- [Node](#node)
- [Pod](#pod)
- [Deployment](#deployment)
- [Service](#service)
- [Ingress](#ingress)
- [ConfigMap](#configmap)
- [Secret](#secret)
- [Persistent Volume (PV) and Persistent Volume Claim (PVC)](#persistent-volume-pv-and-persistent-volume-claim-pvc)
- [Namespace](#namespace)
- [DaemonSet](#daemonset)
- [Job and CronJob](#job-and-cronjob)
- [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler-hpa)
- [StatefulSet](#statefulset)
- [Volume and StorageClass](#volume-and-storageclass)
- [NetworkPolicy](#networkpolicy)
- [Custom Resource Definitions (CRDs)](#custom-resource-definitions-crds)
- [Helm Charts](#helm-charts)

---

This guide provides a detailed overview of Kubernetes concepts, examples, use cases, installation YAML examples, and visual representations of the architecture. 

<a name="cluster"></a>
## Cluster
A **Kubernetes Cluster** is a group of nodes (machines) that host and manage containerized applications. Each cluster has at least one master node, which controls the state of the cluster, and multiple worker nodes.

### Example
A cloud-based cluster with one master node and multiple worker nodes in Google Kubernetes Engine (GKE).

### Use Case
An ideal setup for running a production microservices application with high availability.

#### YAML Structure for Cluster Creation
```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```

---

<a name="node"></a>
## Node
A **Node** is a single machine within a Kubernetes cluster, capable of running one or more pods.

### Example
In a cloud environment, each node could be a virtual machine instance.

### Use Case
Nodes distribute workloads across the infrastructure.

#### YAML Snippet
The node configuration is usually specified by the cluster management tool (e.g., GKE or EKS) rather than in YAML, as it pertains to the infrastructure layer.

---

<a name="pod"></a>
## Pod
A **Pod** is the smallest deployable unit in Kubernetes, which can contain one or more containers.

### Example
A pod running a web server container and a logging container.

### Use Case
Running tightly coupled applications together within the same lifecycle.

#### YAML Structure for Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
spec:
  containers:
  - name: web-server
    image: nginx
  - name: logger
    image: fluentd
```

---

<a name="deployment"></a>
## Deployment
A **Deployment** is a Kubernetes object for managing a group of identical pods. It provides capabilities like rolling updates and rollback.

### Example
A deployment with three replicas of a web server pod.

### Use Case
Maintaining a stable version of an application by managing pod replicas.

#### YAML Structure for Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.4
```

---

<a name="service"></a>
## Service
A **Service** provides a stable endpoint for accessing a set of pods.

### Example
A service that exposes an application to the public internet.

### Use Case
Load balancing traffic to an application across multiple pods.

#### YAML Structure for Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

<a name="ingress"></a>
## Ingress
**Ingress** manages external access to services, typically using HTTP.

### Example
An ingress with rules to route traffic to different services based on URL paths.

### Use Case
Routing `/api` to backend services and `/static` to frontend services on the same domain.

#### YAML Structure for Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /static
        pathType: Prefix
        backend:
          service:
            name: static-service
            port:
              number: 80
```

---

<a name="configmap"></a>
## ConfigMap
A **ConfigMap** stores non-sensitive configuration data.

### Example
A ConfigMap for environment-specific variables.

### Use Case
Keeping configuration separate from the application code.

#### YAML Structure for ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-demo
data:
  DATABASE_URL: "jdbc:mysql://dbhost:3306/mydatabase"
  DEBUG_LEVEL: "3"
```

---

<a name="secret"></a>
## Secret
A **Secret** stores sensitive information, like passwords and tokens.

### Example
A secret containing API keys for external services.

### Use Case
Securing sensitive data without exposing it in the code.

#### YAML Structure for Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-key-secret
type: Opaque
data:
  api-key: c29tZS1yYW5kb20ta2V5Cg==
```

---

<a name="persistent-volume-pv-and-persistent-volume-claim-pvc"></a>
## Persistent Volume (PV) and Persistent Volume Claim (PVC)
A **Persistent Volume** provides storage, while a **Persistent Volume Claim** is a request for that storage.

### Example
A PV representing cloud disk storage and a PVC used by a database pod.

### Use Case
Ensuring data persistence for applications that require durable storage.

#### YAML Structure for PV and PVC
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```

---

<a name="namespace"></a>
## Namespace
A **Namespace** is a way to divide cluster resources among multiple users.

### Example
Separate namespaces for development, staging, and production environments.

### Use Case
Organizing resources for different environments or teams.

#### YAML Structure for Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-environment
```

---

<a name="daemonset"></a>
## DaemonSet
A **DaemonSet** ensures that specific pods run on each node.

### Example
Running a logging agent on every node in the cluster.

### Use Case
Managing node-wide services like logging or monitoring.

#### YAML Structure for DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logger
spec:
  selector:
    matchLabels:
      name: logger
  template:
    metadata:
      labels:
        name: logger
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd
```

---

<a name="job-and-cronjob"></a>
## Job and CronJob
**Job** runs a one-time task, while **CronJob** schedules a recurring task.

### Example
A Job to back up a database once and a CronJob to clean up logs daily.

### Use Case
Automating scheduled tasks or batch processing.

#### YAML Structure for Job and CronJob
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool
      restartPolicy: OnFailure

---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: daily-cleanup
spec:
  schedule: "0 0 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: cleanup-tool
          restartPolicy: OnFailure
```

---

<a name="horizontal-pod-autoscaler-hpa"></a>
## Horizontal Pod Autoscaler (HPA)
The **HPA** scales pod replicas automatically based on resource utilization.

### Example
Scaling web server pods based on CPU usage.

### Use Case
Scaling applications to handle variable workloads.

#### YAML Structure for HPA
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTarget

Ref:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

---

<a name="statefulset"></a>
## StatefulSet
A **StatefulSet** manages stateful applications with unique and persistent storage.

### Example
A StatefulSet for a MongoDB replica set.

### Use Case
Managing stateful applications where each instance requires its own storage.

#### YAML Structure for StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
        - containerPort: 27017
          name: mongo
```

---

<a name="volume-and-storageclass"></a>
## Volume and StorageClass
**Volumes** provide storage to pods, while **StorageClass** defines types of storage.

### Example
SSD-based storage for a database application.

### Use Case
Choosing storage for applications based on performance requirements.

#### YAML Structure for StorageClass and Volume
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fast-pv
spec:
  storageClassName: fast
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
```

---

<a name="networkpolicy"></a>
## NetworkPolicy
**NetworkPolicy** defines network access rules between pods.

### Example
Restricting access to specific pods.

### Use Case
Securing inter-service communication.

#### YAML Structure for NetworkPolicy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress: []
  egress: []
```

---

<a name="custom-resource-definitions-crds"></a>
## Custom Resource Definitions (CRDs)
**CRDs** extend Kubernetes by allowing users to create custom resources.

### Example
A custom resource to manage database schema migrations.

### Use Case
Adding domain-specific resources for applications.

#### YAML Structure for CRD
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: migrations.mycompany.com
spec:
  group: mycompany.com
  versions:
  - name: v1
    served: true
    storage: true
  scope: Namespaced
  names:
    plural: migrations
    singular: migration
    kind: Migration
```

---

<a name="helm-charts"></a>
## Helm Charts
**Helm** is a package manager for Kubernetes, and **Helm Charts** are pre-configured packages.

### Example
A Helm chart for deploying a WordPress site.

### Use Case
Simplifying application deployment and management.

#### Example Helm Command
```bash
helm install my-wordpress bitnami/wordpress
```

---

These concepts are essential for understanding and operating Kubernetes in production, allowing you to deploy, manage, and scale applications effectively. Each example and YAML configuration highlights the power and flexibility Kubernetes offers in a variety of scenarios.