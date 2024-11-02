---
title: Deep Dive into Kubernetes Ingress
date: 2024-10-31 13:20:57
categories:
- Deep Dive
- Kubernetes
tags:
- Deep Dive
- Kubernetes
---


- [What is Kubernetes Ingress?](#what-is-kubernetes-ingress)
  - [Ingress Types](#ingress-types)
- [Ingress Controllers](#ingress-controllers)
- [Ingress Rules and Annotations](#ingress-rules-and-annotations)
  - [Example of an Ingress Rule](#example-of-an-ingress-rule)
- [Ingress Architecture Diagram](#ingress-architecture-diagram)
  - [Ingress with Path-Based Routing](#ingress-with-path-based-routing)

---

In Kubernetes, **Ingress** is a crucial component that manages external access to services within a cluster, typically via HTTP or HTTPS. It provides a centralized entry point for external traffic, applying routing rules to direct requests to appropriate services. This article explores the architecture and responsibilities of Kubernetes Ingress, how it interacts with other components, and provides visual architecture diagrams.

---

<a name="kubernetes-ingress"></a>
## What is Kubernetes Ingress?

**Ingress** in Kubernetes is an API object that manages external access to services, usually HTTP. Instead of exposing each service individually through a **Service** object of type `LoadBalancer` or `NodePort`, Ingress offers a central route, managing traffic with a single IP address or DNS name for the entire cluster. Key benefits include:

- **Centralized Traffic Management**: Ingress consolidates traffic through a single point of entry, allowing easy route management.
- **Path-Based Routing**: Requests can be directed to specific services based on request paths or hostnames.
- **SSL Termination**: Ingress can terminate SSL/TLS connections, ensuring secure HTTPS access.

### Ingress Types
Kubernetes supports two main Ingress types:
- **Simple Ingress**: For straightforward routing to backend services based on path or hostname.
- **Advanced Ingress**: With more complex routing rules, integrating SSL termination, URL rewrites, and custom filtering.

---

<a name="ingress-controllers"></a>
## Ingress Controllers

An **Ingress Controller** is the implementation that enforces Ingress rules, usually deployed as a Pod within the cluster. Popular Ingress controllers include **NGINX**, **HAProxy**, **Traefik**, and **Istio**. The Ingress Controller is responsible for:

1. **Monitoring Ingress Resources**: Watching for any changes in Ingress resources and updating routing rules accordingly.
2. **Load Balancing and SSL Termination**: Providing efficient load balancing for HTTP/HTTPS requests.
3. **Proxying and Forwarding**: Accepting requests on behalf of the Ingress and forwarding them to the correct Service and Pod.

The Ingress Controller must be installed in the cluster to interpret and apply Ingress resources, as Kubernetes does not ship with a default controller.

---

<a name="ingress-rules"></a>
## Ingress Rules and Annotations

Ingress rules define the routing policy for external traffic. These rules can direct traffic based on URL paths or hostnames, and each rule specifies the `Service` and `Port` the request should route to. Annotations add configurations like timeouts, request limits, and additional routing options.

### Example of an Ingress Rule

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

In this example, requests to `example.com/app` are routed to the `app-service` on port `80`.

---

<a name="architecture-diagram"></a>
## Ingress Architecture Diagram

### Ingress with Path-Based Routing

```plaintext
+-----------------------------+
|        External User        |
+-----------------------------+
              |
              v
+-----------------------------+
|      Ingress Controller     |        <-- Manages Ingress Rules
+-----------------------------+
              |
      +-------+------+--------+
      |              |        |
      v              v        v
+-----------+    +-----------+   +-----------+
| Service A |    | Service B |   | Service C |
+-----------+    +-----------+   +-----------+
      |              |                |
      v              v                v
+-----------+    +-----------+   +-----------+
| Pod A1    |    | Pod B1    |   | Pod C1    |
+-----------+    +-----------+   +-----------+
```

In this diagram:
- **Ingress Controller** routes based on rules that direct traffic to Services.
- **Services** are abstractions that select Pods through labels.
- **Pods** run the actual applications and receive traffic forwarded by the Ingress Controller.

This setup allows centralized and managed external access to cluster resources.

---

By centralizing access with Ingress, Kubernetes can handle complex routing configurations, ensuring that traffic flows efficiently and securely across the cluster.
