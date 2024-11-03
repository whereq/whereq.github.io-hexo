---
title: Best Practices of Linux System Performance Analysis and Tuning with CLI
date: 2024-11-03 12:06:47
categories:
- Best Practices
- System
- Performance
- Troubleshooting
- CLI
tags:
- Best Practices
- System
- Performance
- Troubleshooting
- CLI
---

- [Introduction](#introduction)
- [Basic System Monitoring](#basic-system-monitoring)
  - [ Check CPU Usage](#-check-cpu-usage)
  - [ Check Memory Usage](#-check-memory-usage)
  - [ Check Disk I/O](#-check-disk-io)
  - [ Check Network Traffic](#-check-network-traffic)
- [Process Monitoring](#process-monitoring)
  - [ List Running Processes](#-list-running-processes)
  - [ Check Process Status](#-check-process-status)
  - [ Kill a Process](#-kill-a-process)
- [Advanced Performance Tuning](#advanced-performance-tuning)
  - [ Top](#-top)
  - [ Htop](#-htop)
  - [ Iotop](#-iotop)
  - [ Sar](#-sar)
- [Troubleshooting Real Production Issues](#troubleshooting-real-production-issues)
  - [ High CPU Usage](#-high-cpu-usage)
  - [ High Memory Usage](#-high-memory-usage)
  - [ High Disk I/O](#-high-disk-io)
  - [ Network Latency](#-network-latency)
- [Conclusion](#conclusion)
- [References](#references)

---

<a name="introduction"></a>
## Introduction

Monitoring and tuning Linux system performance is crucial for maintaining the health and efficiency of your infrastructure. Command line tools like `top`, `htop`, `iotop`, `sar`, `ps`, `free`, `vmstat`, and `netstat` provide powerful capabilities for analyzing and optimizing system performance. This article provides a comprehensive guide to the best practices of using these tools, covering various scenarios such as checking CPU and memory usage, monitoring processes, and troubleshooting real production issues.

---

<a name="basic-system-monitoring"></a>
## Basic System Monitoring

### <a name="check-cpu-usage"></a> Check CPU Usage

To check CPU usage, use the `top` command:

```bash
top
```

This command displays real-time CPU and memory usage, along with a list of running processes.

### <a name="check-memory-usage"></a> Check Memory Usage

To check memory usage, use the `free` command:

```bash
free -h
```

This command displays the total, used, and free memory in a human-readable format.

### <a name="check-disk-io"></a> Check Disk I/O

To check disk I/O, use the `iostat` command:

```bash
iostat -x 1
```

This command displays detailed I/O statistics, including read and write rates.

### <a name="check-network-traffic"></a> Check Network Traffic

To check network traffic, use the `iftop` command:

```bash
iftop -i eth0
```

This command displays real-time network traffic on the `eth0` interface.

---

<a name="process-monitoring"></a>
## Process Monitoring

### <a name="list-running-processes"></a> List Running Processes

To list all running processes, use the `ps` command:

```bash
ps aux
```

This command displays a detailed list of all running processes.

### <a name="check-process-status"></a> Check Process Status

To check the status of a specific process, use the `ps` command with the process ID (PID):

```bash
ps -p <PID> -o pid,ppid,cmd,%cpu,%mem,state
```

This command displays the status of the process with the specified PID.

### <a name="kill-a-process"></a> Kill a Process

To kill a process, use the `kill` command with the process ID (PID):

```bash
kill <PID>
```

This command sends a termination signal to the process with the specified PID.

---

<a name="advanced-performance-tuning"></a>
## Advanced Performance Tuning

### <a name="top"></a> Top

To monitor system performance in real-time, use the `top` command:

```bash
top
```

This command provides a dynamic view of system resources and running processes.

### <a name="htop"></a> Htop

To monitor system performance with a more user-friendly interface, use `htop`:

```bash
htop
```

This command provides a colorful and interactive view of system resources and running processes.

### <a name="iotop"></a> Iotop

To monitor disk I/O in real-time, use the `iotop` command:

```bash
iotop
```

This command displays real-time disk I/O statistics for running processes.

### <a name="sar"></a> Sar

To collect and report system activity, use the `sar` command:

```bash
sar -u 1 5
```

This command collects CPU usage statistics every second for 5 seconds.

---

<a name="troubleshooting-real-production-issues"></a>
## Troubleshooting Real Production Issues

### <a name="high-cpu-usage"></a> High CPU Usage

To identify processes causing high CPU usage, use `top` or `htop`:

```bash
top
```

Sort processes by CPU usage by pressing `P`.

### <a name="high-memory-usage"></a> High Memory Usage

To identify processes causing high memory usage, use `top` or `htop`:

```bash
top
```

Sort processes by memory usage by pressing `M`.

### <a name="high-disk-io"></a> High Disk I/O

To identify processes causing high disk I/O, use `iotop`:

```bash
iotop
```

This command displays real-time disk I/O statistics for running processes.

### <a name="network-latency"></a> Network Latency

To diagnose network latency, use `ping` and `traceroute`:

```bash
ping google.com
traceroute google.com
```

These commands help identify network latency and routing issues.

---

<a name="conclusion"></a>
## Conclusion

Monitoring and tuning Linux system performance is essential for maintaining the health and efficiency of your infrastructure. By mastering the various options and scenarios covered in this article, you can effectively use command line tools like `top`, `htop`, `iotop`, `sar`, `ps`, `free`, `vmstat`, and `netstat` to analyze and optimize system performance.

---

## References

1. [top Man Page](https://man7.org/linux/man-pages/man1/top.1.html)
2. [htop Man Page](https://hisham.hm/htop/index.php?page=documentation)
3. [iotop Man Page](http://www.ex-parrot.com/pdw/iftop/iftop-man.html)
4. [sar Man Page](https://man7.org/linux/man-pages/man1/sar.1.html)
5. [ps Man Page](https://man7.org/linux/man-pages/man1/ps.1.html)
6. [free Man Page](https://man7.org/linux/man-pages/man1/free.1.html)
7. [iostat Man Page](https://man7.org/linux/man-pages/man8/iostat.8.html)
8. [iftop Man Page](http://www.ex-parrot.com/pdw/iftop/iftop-man.html)
9. [ping Man Page](https://man7.org/linux/man-pages/man8/ping.8.html)
10. [traceroute Man Page](https://man7.org/linux/man-pages/man8/traceroute.8.html)

By following these best practices, you can leverage these system performance tools to their full potential and efficiently manage your monitoring and tuning tasks.