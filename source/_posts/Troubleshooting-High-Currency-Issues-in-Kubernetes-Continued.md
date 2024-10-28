---
title: Troubleshooting High Currency Issues in Kubernetes - Continued
date: 2024-10-27 21:30:37
categories:
- Kubernetes
- Troubleshooting
tags:
- Kubernetes
- Troubleshooting
---


- [Pod Restart `CrashLoopBackOff` Loop Due to High Concurrency (Continued)](#pod-restart-crashloopbackoff-loop-due-to-high-concurrency-continued)
  - [Introduction](#introduction)
  - [Problem Background](#problem-background)
  - [Cause Analysis](#cause-analysis)
  - [Verification of Previous Fixes](#verification-of-previous-fixes)
  - [Exploring Additional Causes](#exploring-additional-causes)
  - [Identifying File Descriptor Limits](#identifying-file-descriptor-limits)
  - [Solution](#solution)
  - [Conclusion](#conclusion)

---

# Pod Restart `CrashLoopBackOff` Loop Due to High Concurrency (Continued)

---

<a name="introduction"></a>
## Introduction

Following the previous issue, the environment experienced another instance of Harbor and Calico pods restarting due to failed health checks. Additionally, using `kubectl` commands to enter the pods was extremely slow or timed out. This article continues the troubleshooting process to identify and resolve the root cause.

---

<a name="problem-background"></a>
## Problem Background

After a period, the environment again faced issues with Harbor and Calico pods restarting due to failed health checks. Using `kubectl` commands to enter the pods was slow or timed out.

```bash
[root@node01 ~]# kubectl exec -it -n system node1-59c9475bc6-zkhq5 bash
^
```

---

<a name="cause-analysis"></a>
## Cause Analysis

The root cause of the repeated restarts was previously identified as health check timeouts, with TCP connections stuck in the `SYN_SENT` phase.

```bash
[root@node01 ~]# netstat -anp | grep 23380
tcp        0      0 127.0.0.1:23380         0.0.0.0:*               LISTEN      38914/kubelet
tcp        0      0 127.0.0.1:38983         127.0.0.1:23380         SYN_SENT    -
```

---

<a name="verification-of-previous-fixes"></a>
## Verification of Previous Fixes

First, check if the previous kernel parameter adjustments are still in place.

```bash
[root@node01 ~]# cat /etc/sysctl.conf
net.ipv4.tcp_max_syn_backlog = 32768
net.core.somaxconn = 32768
```

Verify if the changes have taken effect.

```bash
[root@node01 ~]# ss -lnt
State      Recv-Q   Send-Q     Local Address:Port      Peer Address:Port          
LISTEN     0        32768      127.0.0.1:23380         *:*
```

Despite the parameters being correctly set, the issue persisted.

---

<a name="exploring-additional-causes"></a>
## Exploring Additional Causes

Given the global impact of the issue, it was likely due to a system-level performance bottleneck. High business load affects CPU, memory, disk, and connection counts. Since the connections were long-lived, the file descriptor limit could be a factor.

---

<a name="identifying-file-descriptor-limits"></a>
## Identifying File Descriptor Limits

Check the number of open file descriptors.

```bash
[root@node01 ~]# lsof -p 45775 | wc -l
17974

[root@node01 ~]# lsof -p 45775 | grep "sock" | wc -l
12051
```

The process had over 10,000 open sockets, exceeding the default limit of 1024.

```bash
[root@node01 ~]# ulimit -n
1024
```

Despite exceeding the limit, the business pod did not report "too many open files" errors.

```bash
[root@node01 ~]# kubectl logs -n system node1-59c9475bc6-zkhq5
start config
...
```

---

<a name="solution"></a>
## Solution

Temporarily increase the file descriptor limit.

```bash
[root@node01 ~]# ulimit -n 65535
[root@node01 ~]# ulimit -n
65535
```

After the adjustment, `kubectl` commands responded normally, and the `SYN_SENT` phase issue was resolved.

```bash
[root@node01 ~]# kubectl exec -it -n system node1-59c9475bc6-zkhq5 bash
[root@node1-59c9475bc6-zkhq5]# exit
[root@node01 ~]# kubectl exec -it -n system node1-59c9475bc6-zkhq5 bash
[root@node1-59c9475bc6-zkhq5]# exit
[root@node01 ~]# kubectl exec -it -n system node1-59c9475bc6-zkhq5 bash
[root@node1-59c9475bc6-zkhq5]# exit

[root@node01 ~]# netstat -anp | grep 23380
tcp        0      0 127.0.0.1:23380         0.0.0.0:*               LISTEN      38914/kubelet
tcp        0      0 127.0.0.1:56369         127.0.0.1:23380         TIME_WAIT   -
tcp        0      0 127.0.0.1:23380         127.0.0.1:57601         TIME_WAIT   -
tcp        0      0 127.0.0.1:23380         127.0.0.1:57479         TIME_WAIT   -
```

---

<a name="conclusion"></a>
## Conclusion

This case study highlights the importance of considering system-level parameters, such as file descriptor limits, in high-concurrency scenarios. Adjusting these parameters can resolve performance bottlenecks and ensure stable operation of Kubernetes environments. For environments with high business load, it is strongly recommended to perform comprehensive operating system-level performance optimizations.