---
title: Deep Dive into Core Kubernetes Components
date: 2024-10-31 12:29:28
categories:
- Deep Dive
- Kubernetes
tags:
- Deep Dive
- Kubernetes
---

- [API Server](#api-server)
- [etcd](#etcd)
- [Controller Manager](#controller-manager)
- [Scheduler](#scheduler)
- [Kubelet](#kubelet)
- [Kube Proxy](#kube-proxy)
- [Communication Flow](#communication-flow)

---

This article dives deep into the core components of Kubernetes, explaining their roles, functions, and interconnections. Textual diagrams illustrate the architecture to offer a clearer understanding of each component's interactions.

---

<a name="api-server"></a>
## API Server

The **API Server** is the core entry point for all Kubernetes interactions. It exposes the Kubernetes API, validates requests, and interacts with other components. Key functionalities include:

- **Authentication**: Verifies the identity of incoming requests.
- **Authorization**: Manages RBAC (Role-Based Access Control) policies.
- **Admission Control**: Applies constraints and transformations to objects at request time.
- **API Requests**: Handles CRUD (Create, Read, Update, Delete) requests on Kubernetes objects.

**Diagram:**
```
+----------------+
|                |
|    API Server  |
|                |
+----------------+
       |
       v
+----------------+
|                |
|      etcd      |
|                |
+----------------+
```

---

<a name="etcd"></a>
## etcd

**etcd** is a highly available key-value store used by Kubernetes to persist the cluster state. It maintains the configurations and states of each component and is essential for recovering cluster state in case of a restart. Each component retrieves cluster state through the API Server, which interfaces with etcd.

**Diagram:**
```
+----------------+           +----------------+
|                |           |                |
|    API Server  | <-------> |      etcd      |
|                |           |                |
+----------------+           +----------------+
```

---

<a name="controller-manager"></a>
## Controller Manager

The **Controller Manager** includes various controllers (e.g., Node, Replication, Endpoint) that regulate the state of cluster resources. Each controller reconciles the cluster's actual state with the desired state. When discrepancies arise, controllers initiate adjustments (e.g., scaling replicas).

**Diagram:**
```
+----------------+
|                |
| Controller     |
| Manager        |
|                |
+----------------+
       |
       v
+----------------+
|                |
|    API Server  |
|                |
+----------------+
```

---

<a name="scheduler"></a>
## Scheduler

The **Scheduler** assigns Pods to Nodes based on resource needs and constraints. The Scheduler evaluates factors like CPU, memory, and affinity rules before assigning Pods, ensuring optimal workload distribution.

**Diagram:**
```
+----------------+
|                |
|   Scheduler    |
|                |
+----------------+
       |
       v
+----------------+
|                |
|    API Server  |
|                |
+----------------+
```

---

<a name="kubelet"></a>
## Kubelet

The **Kubelet** runs on each Node and is responsible for managing container instances. It communicates with the API Server to receive Pod specifications, ensuring they’re running as defined. It monitors containers, collects resource usage, and sends reports back to the API Server.

**Diagram:**
```
+----------------+     +----------------+
|                |     |                |
|     Kubelet    | <-> |  API Server    |
|                |     |                |
+----------------+     +----------------+
```

---

<a name="kube-proxy"></a>
## Kube Proxy

**Kube Proxy** manages network communication across Nodes in the cluster, implementing Kubernetes networking policies. It facilitates communication within Pods by maintaining network rules and forwarding requests as needed. 

**Diagram:**
```
+----------------+     +----------------+
|                |     |                |
|   Kube Proxy   | <-> |   Kubelet      |
|                |     |                |
+----------------+     +----------------+
```

---

<a name="communication-flow"></a>
## Communication Flow

In Kubernetes, components communicate through a centralized and secure API Server, with `etcd` acting as the persistent store. The workflow for scheduling, updating, and scaling applications includes:

1. **User Command**: Users issue requests to the API Server.
2. **Scheduler**: Schedules new Pods.
3. **Controller Manager**: Enforces and monitors the cluster state.
4. **Kubelet and Proxy**: Enforce changes and maintain container and network policies.

**Overall Diagram:**
```
+----------------+            +----------------+
|                |            |                |
|    API Server  | <--------> |      etcd      |
|                |            |                |
+----------------+            +----------------+
       |                            |
       |                            |
       v                            v
+----------------+         +----------------+
|                |         |                |
| Controller     |         |   Scheduler    |
| Manager        |         |                |
+----------------+         +----------------+
       |                            |
       v                            v
+----------------+         +----------------+
|                |         |                |
|    Kubelet     |         |   Kube Proxy   |
|                |         |                |
+----------------+         +----------------+
```

---

Each Kubernetes component plays a specific role in cluster operations, and their communication ensures the cluster maintains its desired state and remains responsive to changing demands.
