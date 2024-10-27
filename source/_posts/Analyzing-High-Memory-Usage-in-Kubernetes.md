---
title: Analyzing High Memory Usage in Kubernetes
date: 2024-10-27 19:28:43
categories:
- Kubernetes
tags:
- Kubernetes
---

- [Introduction](#introduction)
- [Problem Background](#problem-background)
- [Cause Analysis](#cause-analysis)
  - [Using `docker stats`](#using-docker-stats)
- [Verification of Memory Calculation](#verification-of-memory-calculation)
  - [Verifying with `memory_stats`](#verifying-with-memory_stats)
- [Understanding Cache Usage](#understanding-cache-usage)
- [Solution](#solution)
- [Conclusion](#conclusion)

---

<a name="introduction"></a>
## Introduction

In Kubernetes environments, monitoring resource usage is crucial for maintaining optimal performance and resource allocation. This article delves into a specific case where a user observed high memory usage by a Harbor pod on one of the nodes, leading to a discrepancy between `kubectl top` and `docker stats` results. We will analyze the cause, verify the memory calculation methods, and provide a solution to accurately monitor memory usage.

---

<a name="problem-background"></a>
## Problem Background

A user observed that one of the nodes in their Kubernetes cluster showed high memory usage by a Harbor pod, approximately 3.7G, which seemed unusually high compared to other pods.

```bash
[root@node02 ~]# kubectl get node -owide
NAME    STATUS   ROLES    AGE   VERSION    INTERNAL-IP   EXTERNAL-IP    
node01  Ready    master   10d   v1.15.12   100.1.0.10   <none>   
node02  Ready    master   12d   v1.15.12   100.1.0.11   <none>  
node03  Ready    master   10d   v1.15.12   100.1.0.12   <none> 

[root@node02 ~]# kubectl top pod -A | grep harbor
kube-system         harbor-master1-sxg2l                   15m           150Mi
kube-system         harbor-master2-ncvb8                   8m            3781Mi
kube-system         harbor-master3-2gdsn                   14m           227Mi
```

---

<a name="cause-analysis"></a>
## Cause Analysis

To understand the memory usage, the user compared the results from `kubectl top` and `docker stats`. Theoretically, `docker stats` should provide a more accurate result.

### Using `docker stats`

```bash
[root@node02 ~]# docker stats | grep harbor
CONTAINER ID        NAME                                  CPU %    MEM USAGE / LIMIT     MEM %
10a230bee3c7        k8s_nginx_harbor-master2-xxx          0.02%    14.15MiB / 94.26GiB   0.01%
6ba14a04fd77        k8s_harbor-portal_harbor-master2-xxx  0.01%    13.73MiB / 94.26GiB   0.01%
324413da20a9        k8s_harbor-jobservice_harbor-master2-xxx  0.11%    21.54MiB / 94.26GiB   0.02%
d880b61cf4cb        k8s_harbor-core_harbor-master2-xxx    0.12%    33.2MiB / 94.26GiB    0.03%
186c064d0930        k8s_harbor-registryctl_harbor-master2-xxx 0.01%    8.34MiB / 94.26GiB    0.01%
52a50204a962        k8s_harbor-registry_harbor-master2-xxx 0.06%    29.99MiB / 94.26GiB   0.03%
86031ddd0314        k8s_harbor-redis_harbor-master2-xxx   0.14%    11.51MiB / 94.26GiB   0.01%
6366207680f2        k8s_harbor-database_harbor-master2-xxx 0.45%    8.859MiB / 94.26GiB   0.01%
```

The total memory usage from `docker stats` was around 140M, significantly lower than the 3.7G reported by `kubectl top`.

---

<a name="verification-of-memory-calculation"></a>
## Verification of Memory Calculation

The discrepancy arises from the different methods used by `kubectl top` and `docker stats` to calculate memory usage.

- **`kubectl top` Calculation**: `memory.usage_in_bytes - inactive_file`
- **`docker stats` Calculation**: `memory.usage_in_bytes - cache`

If the cache is large, `kubectl top` will report a higher memory usage.

### Verifying with `memory_stats`

To verify, the user fetched the `memory_stats` data using Docker's API:

```bash
curl -s --unix-socket /var/run/docker.sock http:/v1.24/containers/xxx/stats | jq ."memory_stats"
```

Sample `memory_stats` data:

```json
"memory_stats": {
    "usage": 14913536,
    "max_usage": 15183872,
    "stats": {
      "active_anon": 14835712,
      "active_file": 0,
      "cache": 77824,
      "dirty": 0,
      "hierarchical_memory_limit": 101205622784,
      "hierarchical_memsw_limit": 9223372036854772000,
      "inactive_anon": 4096,
      "inactive_file": 73728,
      ...
}
```

By applying the formulas, the user confirmed that the results from `kubectl top` and `docker stats` were consistent with the expected calculations.

---

<a name="understanding-cache-usage"></a>
## Understanding Cache Usage

The high cache usage in Harbor was primarily due to the `registry` component, which handles Docker image storage and involves extensive read/write operations on image layers.

```json
"memory_stats": {
    "usage": 5845127168,
    "max_usage": 22050988032,
    "stats": {
    "active_anon": 31576064,
    "active_file": 3778052096,
    "cache": 5813551104,
    "dirty": 0,
    "hierarchical_memory_limit": 101205622784,
    "hierarchical_memsw_limit": 9223372036854772000,
    "inactive_anon": 0,
    "inactive_file": 2035499008,
    ...
}
```

---

<a name="solution"></a>
## Solution

To address the discrepancy, the user was informed that `kubectl top` includes cache usage, which can be misleading. The cache, being non-critical, can be reclaimed by the system during memory pressure or manually released. The user was advised to use `docker stats` for a more accurate representation of actual memory usage.

---

<a name="conclusion"></a>
## Conclusion

This case study highlights the importance of understanding the different methods used by Kubernetes and Docker to calculate memory usage. By verifying the calculations and understanding the role of cache, users can make more informed decisions about resource management in their Kubernetes clusters. Using `docker stats` provides a more accurate view of memory usage, especially in scenarios where cache usage is significant.