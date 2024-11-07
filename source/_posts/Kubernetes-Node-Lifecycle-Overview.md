---
title: Kubernetes Node Lifecycle Overview
date: 2024-11-05 21:20:04
categories:
- Kubernetes
tags:
- Kubernetes
---

# Kubernetes Node Lifecycle Overview

A Kubernetes `Node` is a core component with a lifecycle that includes registration, operation, and decommissioning. This article briefly explains key events in the Node lifecycle.

![Kubernetes Node Lifecycle Overview](images/Kubernetes-Node-Lifecycle-Overview/Image-1.png)

### Node Registration

Each node must run `kubelet`, which initiates a registration request to `kube-apiserver`, creating a `node` resource object. `registerNode` (default: true) in the kubelet configuration controls automatic registration. If set to false, nodes must be registered manually.

The `nodename` is determined by:
- The cloud provider, if configured.
- Otherwise, the local hostname, which `--hostname-override` can modify.

### Node Heartbeat Mechanism

The heartbeat includes updating both `.status` and a corresponding `lease` object for the node. `nodeStatusUpdateFrequency` (default: 10 seconds) triggers kubelet to update `.status` on changes or every 10 seconds. Additionally, each node maintains a `lease` object in `kube-node-lease` with an update frequency set by `nodeLeaseDurationSeconds` (default: 40 seconds) multiplied by 0.25, or 10 seconds.

### Node Health Monitoring

The `node-controller` in `controller-manager` monitors node health. If no issues arise, the node remains active. However, if a node loses connectivity or crashes, `node-controller` will detect this after a period specified by `--node-monitor-grace-period` (default: 40 seconds, increasing to 50 in v1.32). After this period, the node status changes to `Unknown`, and a `Taint` is applied to prevent new pod scheduling. If heartbeat remains absent for 5 more minutes, the node-controller starts eviction of pods and other resources from the node.

A graceful node shutdown follows a similar procedure: marking taints, rescheduling pods, and decommissioning the node.