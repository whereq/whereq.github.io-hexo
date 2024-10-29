---
title: 'Troubleshooting Kubernetes: Pod Stuck in Terminating State After Deletion'
date: 2024-10-28 21:10:13
categories:
- Kubernetes
- Troubleshooting
tags:
- Kubernetes
- Troubleshooting
---

- [Introduction](#introduction)
- [Problem Background](#problem-background)
- [Cause Analysis](#cause-analysis)
  - [ Initial Investigation](#-initial-investigation)
  - [ Kubelet Logs Analysis](#-kubelet-logs-analysis)
    - [Suspicious Point 1: Pod IP Retrieval Failure](#suspicious-point-1-pod-ip-retrieval-failure)
    - [Suspicious Point 2: Unmount Failure](#suspicious-point-2-unmount-failure)
- [Suspicious Points](#suspicious-points)
  - [ Suspicious Point 1: Pod IP Retrieval Failure](#-suspicious-point-1-pod-ip-retrieval-failure)
  - [ Suspicious Point 2: Unmount Failure](#-suspicious-point-2-unmount-failure)
- [Pod Deletion Process](#pod-deletion-process)
- [Verification and Testing](#verification-and-testing)
- [Conclusion](#conclusion)

---

<a name="introduction"></a>
## Introduction

This document provides a detailed troubleshooting guide for a common Kubernetes issue where a Pod remains in the `Terminating` state after being deleted using the `kubectl delete` command. The analysis includes examining logs, identifying suspicious points, and understanding the underlying causes.

---

<a name="problem-background"></a>
## Problem Background

After deleting a business Pod using the `kubectl delete` command, the Pod remained in the `Terminating` state.

---

<a name="cause-analysis"></a>
## Cause Analysis

### <a name="initial-investigation"></a> Initial Investigation

To begin with, the `kubectl describe` command was used to inspect the Pod:

```bash
[root@node1 ~]# kubectl describe pod -n xxx cam1-78b6fc6bc8-cjsw5
```

No obvious anomalies were found in the output.

### <a name="kubelet-logs-analysis"></a> Kubelet Logs Analysis

Next, the kubelet component logs were examined for any anomalies during the Pod deletion process. The following logs were filtered for relevance:

```bash
I0728 16:24:57.339295    9744 kubelet.go:1904] SyncLoop (DELETE, "api"): "cam1-78b6fc6bc8-cjsw5_cam(5c948341-c030-4996-b888-f032577d97b0)"
I0728 16:24:57.339720    9744 kuberuntime_container.go:581] Killing container "docker://a73082a4a9a4cec174bb0d1c256cc11d804d93137551b9bfd3e6fa1522e98589" with 60 second grace period
I0728 16:25:18.259418    9744 kubelet.go:1904] SyncLoop (DELETE, "api"): "cam1-78b6fc6bc8-cjsw5_cam(5c948341-c030-4996-b888-f032577d97b0)"
2021-07-28 16:25:19.247 [INFO][394011] ipam.go 1173: Releasing all IPs with handle 'cam.cam1-78b6fc6bc8-cjsw5'
2021-07-28 16:25:19.254 [INFO][393585] k8s.go 498: Teardown processing complete.
```

#### Suspicious Point 1: Pod IP Retrieval Failure

```bash
W0728 16:25:19.303513    9744 docker_sandbox.go:384] failed to read pod IP from plugin/docker: NetworkPlugin cni failed on the status hook for pod "cam1-78b6fc6bc8-cjsw5_cam": Unexpected command output Device "eth0" does not exist.
 with error: exit status 1
```

#### Suspicious Point 2: Unmount Failure

```bash
E0728 16:25:20.939400    9744 nestedpendingoperations.go:301] Operation for "{volumeName:kubernetes.io/glusterfs/5c948341-c030-4996-b888-f032577d97b0-cam-pv-50g podName:5c948341-c030-4996-b888-f032577d97b0 nodeName:}" failed. No retries permitted until 2021-07-28 16:25:21.439325811 +0800 CST m=+199182.605079651 (durationBeforeRetry 500ms). Error: "UnmountVolume.TearDown failed for volume \"diag-log\" (UniqueName: \"kubernetes.io/glusterfs/5c948341-c030-4996-b888-f032577d97b0-cam-pv-50g\") pod \"5c948341-c030-4996-b888-f032577d97b0\" (UID: \"5c948341-c030-4996-b888-f032577d97b0\") : Unmount failed: exit status 32\nUnmounting arguments: /var/lib/kubelet/pods/5c948341-c030-4996-b888-f032577d97b0/volumes/kubernetes.io~glusterfs/cam-pv-50g\nOutput: umount: /var/lib/kubelet/pods/5c948341-c030-4996-b888-f032577d97b0/volumes/kubernetes.io~glusterfs/cam-pv-50g：Target is busy.\n        (In some case using lsof(8) or fuser(1) can\n find useful information about the process using the device)\n\n"
```

---

<a name="suspicious-points"></a>
## Suspicious Points

### <a name="suspicious-point-1-pod-ip-retrieval-failure"></a> Suspicious Point 1: Pod IP Retrieval Failure

The log indicates a failure to retrieve the Pod IP:

```bash
W0728 16:25:19.303513    9744 docker_sandbox.go:384] failed to read pod IP from plugin/docker: NetworkPlugin cni failed on the status hook for pod "cam1-78b6fc6bc8-cjsw5_cam": Unexpected command output Device "eth0" does not exist.
 with error: exit status 1
```

This error was traced to the `getIP` method in the `docker_sandbox.go` file:

```go
func (ds *dockerService) getIP(podSandboxID string, sandbox *dockertypes.ContainerJSON) string {
    if sandbox.NetworkSettings == nil {
        return ""
    }
    if networkNamespaceMode(sandbox) == runtimeapi.NamespaceMode_NODE {
        // For sandboxes using host network, the shim is not responsible for
        // reporting the IP.
        return ""
    }

    // Don't bother getting IP if the pod is known and networking isn't ready
    ready, ok := ds.getNetworkReady(podSandboxID)
    if ok && !ready {
        return ""
    }

    ip, err := ds.getIPFromPlugin(sandbox)
    if err == nil {
        return ip
    }
    
    if sandbox.NetworkSettings.IPAddress != "" {
        return sandbox.NetworkSettings.IPAddress
    }
    if sandbox.NetworkSettings.GlobalIPv6Address != "" {
        return sandbox.NetworkSettings.GlobalIPv6Address
    }

    // Error log here
    klog.Warningf("failed to read pod IP from plugin/docker: %v", err)
    return ""
}
```

### <a name="suspicious-point-2-unmount-failure"></a> Suspicious Point 2: Unmount Failure

The unmount failure was logged as an ERROR:

```bash
E0728 16:25:20.939400    9744 nestedpendingoperations.go:301] Operation for "{volumeName:kubernetes.io/glusterfs/5c948341-c030-4996-b888-f032577d97b0-cam-pv-50g podName:5c948341-c030-4996-b888-f032577d97b0 nodeName:}" failed. No retries permitted until 2021-07-28 16:25:21.439325811 +0800 CST m=+199182.605079651 (durationBeforeRetry 500ms). Error: "UnmountVolume.TearDown failed for volume \"diag-log\" (UniqueName: \"kubernetes.io/glusterfs/5c948341-c030-4996-b888-f032577d97b0-cam-pv-50g\") pod \"5c948341-c030-4996-b888-f032577d97b0\" (UID: \"5c948341-c030-4996-b888-f032577d97b0\") : Unmount failed: exit status 32\nUnmounting arguments: /var/lib/kubelet/pods/5c948341-c030-4996-b888-f032577d97b0/volumes/kubernetes.io~glusterfs/cam-pv-50g\nOutput: umount: /var/lib/kubelet/pods/5c948341-c030-4996-b888-f032577d97b0/volumes/kubernetes.io~glusterfs/cam-pv-50g：Target is busy.\n        (In some case using lsof(8) or fuser(1) can\n find useful information about the process using the device)\n\n"
```

---

<a name="pod-deletion-process"></a>
## Pod Deletion Process

The deletion of a Pod in Kubernetes follows these steps:

1. **API Server DELETE Call**: The `kubectl delete` command triggers a DELETE request to the Kubernetes API server with a default grace period of 30 seconds.
2. **Update Pod Metadata**: The Pod's metadata is updated with `DeletionTimestamp` and `DeletionGracePeriodSeconds` fields, but the Pod is not yet deleted from etcd.
3. **Kubelet Event Handling**: The kubelet component listens for Pod updates and initiates the `killPod()` method.
4. **Second DELETE Call**: The kubelet makes a second DELETE request to the API server with a grace period of 0 seconds.
5. **Pod Deletion from etcd**: The API server deletes the Pod from etcd.

---

<a name="verification-and-testing"></a>
## Verification and Testing

To verify the deletion process, a test was conducted in a test environment:

```bash
[root@node2 ~]# kubectl delete pod -n xxx testpodrc2-7b749f6c9c-qh68l
pod "testpodrc2-7b749f6c9c-qh68l" deleted
```

Filtered kubelet logs:

```bash
I0730 13:27:31.854178   24588 kubelet.go:1904] SyncLoop (DELETE, "api"): "testpodrc2-7b749f6c9c-qh68l_testpod(85ee282f-a843-4f10-a99c-79d447f83f2a)"
I0730 13:27:31.854511   24588 kuberuntime_container.go:581] Killing container "docker://e2a1cd5f2165e12cf0b46e12f9cd4d656d593f75e85c0de058e0a2f376a5557e" with 30 second grace period
I0730 13:27:32.203167   24588 kubelet.go:1904] SyncLoop (DELETE, "api"): "testpodrc2-7b749f6c9c-qh68l_testpod(85ee282f-a843-4f10-a99c-79d447f83f2a)"
I0730 13:27:32.993294   24588 kubelet.go:1933] SyncLoop (PLEG): "testpodrc2-7b749f6c9c-qh68l_testpod(85ee282f-a843-4f10-a99c-79d447f83f2a)", event: &pleg.PodLifecycleEvent{ID:"85ee282f-a843-4f10-a99c-79d447f83f2a", Type:"ContainerDied", Data:"e2a1cd5f2165e12cf0b46e12f9cd4d656d593f75e85c0de058e0a2f376a5557e"}
I0730 13:27:32.993428   24588 kubelet.go:1933] SyncLoop (PLEG): "testpodrc2-7b749f6c9c-qh68l_testpod(85ee282f-a843-4f10-a99c-79d447f83f2a)", event: &pleg.PodLifecycleEvent{ID:"85ee282f-a843-4f10-a99c-79d447f83f2a", Type:"ContainerDied", Data:"c6a587614976beed0cbb6e5fabf70a2d039eec6c160154fce007fe2bb1ba3b4f"}
I0730 13:27:34.072494   24588 kubelet_pods.go:1090] Killing unwanted pod "testpodrc2-7b749f6c9c-qh68l"
I0730 13:27:40.084182   24588 kubelet.go:1904] SyncLoop (DELETE, "api"): "testpodrc2-7b749f6c9c-qh68l_testpod(85ee282f-a843-4f10-a99c-79d447f83f2a)"
I0730 13:27:40.085735   24588 kubelet.go:1898] SyncLoop (REMOVE, "api"): "testpodrc2-7b749f6c9c-qh68l_testpod(85ee282f-a843-4f10-a99c-79d447f83f2a)"
```

---

<a name="conclusion"></a>
## Conclusion

The analysis revealed that the Pod remained in the `Terminating` state due to a failure in unmounting the GlusterFS volume. The unmount failure was caused by the volume being busy, which prevented the second DELETE
