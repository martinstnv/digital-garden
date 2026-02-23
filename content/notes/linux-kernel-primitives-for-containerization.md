---
title: Linux Kernel Primitives for Containerization
date: 2026-02-23
tags:
  - Linux
  - Kernel
---

# Theory

Containers are standard user-space processes running on a host operating system. They're considered "containers" because the Linux kernel applies isolation boundaries around them. Through this isolation, each process has its own view of system resources, such as processes, networking, and filesystems, creating the illusion that it's running on its own machine. In reality, all containers share the same host kernel.

This model of containerization is primarily enabled by two kernel features: namespaces, which provide resource isolation, and cgroups, which control and limit resource usage. Namespaces define what a process can see. They isolate views of system resources such as process IDs, network interfaces, mount points, and hostnames. Cgroups, on the other hand, define how much of those resources a process can use. They enforce limits and accounting for CPU, memory, disk I/O, and other system resources.

## Process Namespaces

On a typical Linux system, all processes share a single global PID space. A PID namespace changes that by giving a container its own isolated process tree. Inside the container, the main process appears as PID 1, just like the init process on a freshly booted system. From the host's perspective, however, that same process has its actual PID (often a high number) within the global namespace.


## Mount Namespaces

Mount namespaces isolate the filesystem view. Although containers share the same underlying kernel, they can see entirely different filesystem hierarchies. The container's root filesystem is constructed from image layers, which are mounted into the container's mount namespace. As a result, the container has its own view of what is mounted and where, independent of the host.

## Network Namespaces

Network namespaces provide containers with their own network stack, including interfaces, routing tables, and port space. This isolation allows multiple containers to bind to the same port number without conflict, since each operates within its own separate network namespace.

## User Namespaces

User namespaces allow user and group IDs inside a container to be mapped to different IDs on the host. For example, a process running as root (UID 0) inside the container can be mapped to an unprivileged user ID outside the container, reducing the security risk to the host system.

## Control Groups (cgroups)

Control groups, or cgroups, manage and limit resource usage. They enforce constraints on CPU, memory, disk I/O, and other resources, ensuring that one container cannot monopolize system resources and starve other workloads on the same host.

## Containerization Outside Native Linux

Containers are a Linux kernel feature. macOS runs the Darwin kernel, and Windows runs the NT kernel - neither of which provides the native primitives (such as namespaces and cgroups) required for Linux containers.

To bridge this gap, tools like Docker create a lightweight Linux virtual machine in the background. Containers run inside that VM rather than directly on the host operating system.

Because of this additional virtualization layer, there is some performance overhead compared to running containers on native Linux. File system operations are slower since files must cross the VM boundary, and network traffic is also routed through the virtual machine’s networking stack instead of the host’s native one.

# Practice

The following command creates a new PID namespace and starts a shell inside it.

```
user@host:~$ sudo unshare --pid --fork --mount-proc sh
```

Inside the new namespace, the shell sees itself as PID 1, with its own isolated process tree.

```
# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   2380  1408 pts/1    S    21:39   0:00 sh
root           2  0.0  0.0  10716  4096 pts/1    R+   21:39   0:00 ps aux
```

From the host's global PID namespace, the same processes appear with their real PIDs.

```
user@host:~$ ps aux | grep " sh"
root       13285  0.0  0.0  18256  6272 pts/0    S+   21:39   0:00 sudo unshare --pid --fork --mount-proc sh
root       13286  0.0  0.0  18256  2356 pts/1    Ss   21:39   0:00 sudo unshare --pid --fork --mount-proc sh
root       13287  0.0  0.0   5268  1536 pts/1    S    21:39   0:00 unshare --pid --fork --mount-proc sh
root       13288  0.0  0.0   2380  1408 pts/1    S+   21:39   0:00 sh
paralle+   18546  0.0  0.0   6140  2048 pts/2    S+   21:45   0:00 grep --color=auto  sh
```

Each process has a set of namespace references exposed through the /proc filesystem. These entries are the actual namespace handles that the kernel has attached to that specific process.

```
user@host:~$ sudo ls -la /proc/13288/ns/
total 0
dr-x--x--x 2 root root 0 Feb 23 21:42 .
dr-xr-xr-x 9 root root 0 Feb 23 21:39 ..
lrwxrwxrwx 1 root root 0 Feb 23 21:42 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 mnt -> 'mnt:[4026532448]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 net -> 'net:[4026531840]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 pid -> 'pid:[4026532451]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 pid_for_children -> 'pid:[4026532451]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Feb 23 21:42 uts -> 'uts:[4026531838]'
```

This is conceptually similar to how Docker’s exec command uses the setns() system call to place a new process into the existing namespaces of a running container.