---
title: Troubleshooting Pod-to-Pod Communication in Kubernetes
date: 2024-11-02 12:49:45
categories:
- Kubernetes
- Troubleshooting
tags:
- Kubernetes
- Troubleshooting
---

- [Introduction](#introduction)
- [Problem Description](#problem-description)
- [Troubleshooting Steps](#troubleshooting-steps)
  - [Step 1: Capture Network Traffic](#step-1-capture-network-traffic)
  - [Step 2: Verify Service IP](#step-2-verify-service-ip)
  - [Step 3: Check DNS Resolution](#step-3-check-dns-resolution)
  - [Step 4: Investigate Service IP](#step-4-investigate-service-ip)
  - [Step 5: Verify Outgoing Traffic from Pod](#step-5-verify-outgoing-traffic-from-pod)
  - [Step 6: Simulate Traffic from Pod](#step-6-simulate-traffic-from-pod)
  - [Step 7: Restart Pod](#step-7-restart-pod)
  - [Step 8: Identify Java DNS Caching](#step-8-identify-java-dns-caching)
- [Solution](#solution)
- [Conclusion](#conclusion)

---

<a name="introduction"></a>
## Introduction

In a K8S cluster, PodA is experiencing issues when using a service name to access PodB. PodA is on `node1`, and PodB is on `node2`. This article will walk through the troubleshooting steps to identify and resolve the issue.

<a name="problem-description"></a>
## Problem Description

PodA is unable to access PodB using the service name, and the request is failing. PodA is on `node1`, and PodB is on `node2`.

<a name="troubleshooting-steps"></a>
## Troubleshooting Steps

<a name="step-1-capture-network-traffic"></a>
### Step 1: Capture Network Traffic

First, use `tcpdump` to capture network traffic and observe if there are any anomalies:

```bash
[root@node1 ~]# tcpdump -n -i ens192 port 50300  
...  
13:48:17.630335 IP 177.177.176.150.distinct -> 10.96.22.136.50300: UDP, length 214  
13:48:17.630407 IP 192.168.7.21.distinct -> 10.96.22.136.50300: UDP, length 214  
...
```

From the captured packets, the source address and port are `177.177.176.150:50901`, and the destination address and port are `10.96.22.136:50300`. The destination IP `10.96.22.136` is the IP address obtained by PodA when using the `server-svc` service name.

<a name="step-2-verify-service-ip"></a>
### Step 2: Verify Service IP

Check if the IP address `10.96.22.136` is correct:

```bash
[root@node1 ~]# kubectl get pod -A -owide|grep server  
ss  server-xxx-xxx  1/1  Running 0 20h  177.177.176.150  node1  
ss  server-xxx-xxx  1/1  Running 0 20h  177.177.254.245  node2  
ss  server-xxx-xxx  1/1  Running 0 20h  177.177.18.152   node3  
```

```bash
[root@node1 ~]# kubectl get svc -A -owide|grep server  
ss  server-svc  ClusterIP  10.96.182.195 <none>  50300/UDP  
```

The source IP is correct, but the destination IP does not match the expected `server-svc` IP `10.96.182.195`.

<a name="step-3-check-dns-resolution"></a>
### Step 3: Check DNS Resolution

Since K8S uses `CoreDNS` for service discovery starting from v1.13, verify if the DNS resolution is correct inside PodA:

```bash
[root@node1 ~]# kubectl exec -it -n ss server-xxx-xxx -- cat /etc/resolve.conf  
nameserver 10.96.0.10  
search ss.svc.cluster.local svc.cluster.local cluster.local  
options ndots:5  
  
[root@node1 ~]# kubectl exec -it -n ss server-xxx-xxx -- nslookup server-svc  
Server: 10.96.0.10  
  
Name: ss  
Address: 10.96.182.195
```

The DNS resolution is correct, and PodA can resolve `server-svc` to `10.96.182.195`.

<a name="step-4-investigate-service-ip"></a>
### Step 4: Investigate Service IP

Check if `10.96.22.136` is associated with any other service, iptables rules, or routes:

```bash
[root@node1 ~]# kubectl get svc -A -owide|grep 10.96.22.136  
  
[root@node1 ~]# iptables-save|grep 10.96.22.136  
  
[root@node1 ~]# ip route|grep 10.96.22.136  
```

The IP `10.96.22.136` does not exist in the cluster.

<a name="step-5-verify-outgoing-traffic-from-pod"></a>
### Step 5: Verify Outgoing Traffic from Pod

Check the destination IP when the traffic leaves PodA:

```bash
[root@node1 ~]# ip route|grep 177.177.176.150  
177.177.176.150 dev cali9afa4438787 scope link  
  
[root@node1 ~]# tcpdump -n -i cali9afa4438787 port 50300  
...  
14:16:40.821511 IP 177.177.176.150.50902 ->  10.96.22.136.50300:  UDP, length 214  
...  
```

The destination IP is incorrect when leaving PodA.

<a name="step-6-simulate-traffic-from-pod"></a>
### Step 6: Simulate Traffic from Pod

Simulate a UDP packet from PodA using `nc` to verify if the destination IP is correct:

```bash
[root@node1 ~]# kubectl exec -it -n ss server-xxx-xxx -- echo “test” | nc -u server-svc 50300 -p 9999  
  
[root@node1 ~]# tcpdump -n -i cali9afa4438787 port 50300  
...  
15:46:45.871580 IP 177.177.176.150.50902 ->  10.96.182.195.50300:  UDP, length 54  
...  
```

The simulated traffic has the correct destination IP.

<a name="step-7-restart-pod"></a>
### Step 7: Restart Pod

Restart PodA to see if the issue is resolved:

```bash
[root@node1 ~]# docker ps |grep server-xxx-xxx | grep -v POD |awk '{print $1}' |xargs docker restart  
  
[root@node1 ~]# tcpdump -n -i ens192 port 50300  
...  
15:58:17.150535 IP 177.177.176.150.distinct -> 10.96.182.195.50300:  UDP, length 214  
15:58:17.150607 IP 192.168.7.21.distinct  ->  10.96.182.195.50300:   UDP, length 214  
...  
```

The issue is resolved after restarting PodA.

<a name="step-8-identify-java-dns-caching"></a>
### Step 8: Identify Java DNS Caching

Investigate if Java DNS caching is causing the issue:

> In Java, DNS caching is used to improve performance by storing DNS lookup results. When `InetAddress` is used to resolve a domain name, the result is cached by the JVM. This cache can cause stale DNS entries to be used.

To resolve this, adjust the DNS caching settings:

1. Set the `networkaddress.cache.ttl` property in the Java security settings.
2. Modify the `java.security` file to set the `networkaddress.cache.negative.ttl` property.

<a name="solution"></a>
## Solution

Adjust the DNS caching settings in the business application based on the business scenario.

<a name="conclusion"></a>
## Conclusion

This article detailed the troubleshooting steps to identify and resolve an issue where PodA was unable to access PodB using a service name in a K8S cluster. The root cause was identified as Java DNS caching, and the issue was resolved by adjusting the DNS caching settings.