---
layout: default
usemathjax: true
permalink: /cc
---

# Cloud Computing

## Virtualization

- **Generally**: Virtual representation of real resources
- **OS i.e. Virtual Machines**: 
  - virtualizing **all** essential hardware components support installing guest OS

VMs require **hypervisors**, the software that runs on host machine that

- creates multiple isolated virtual **operating environments with isolated resources**
- **Type 1**: built on top of host's hardware (bare metal), no host OS needed
  - HyperV
  - VMware ESXi
  - AWS Nitro
- **Type 2**: built on top of host's OS
  - VirtualBox
  - VMware Player/Workstation
- Hybrid: can virtualize both hardware and OS resources.
  - KVM
  - bhyve

### KVM

aka Kernel-based Virtual Machine.

Kernel **becomes** a hypervisor that can manage guest VMs.

![KVM description](/notes-blog/assets/img/cc/kvm.png)

Application

## Containers

- Engine 
- Payload can be encapsulated 
  - lightweight, portable, self-sufficient container
- Standard API
- Runs consistently on any HW

### Images

**Image** = (Abstract) Box of source code + dependencies + libraries.

- Application
- Dependencies
- User-space libraries (e.g. glibc) for switching from user-space to kernel-space.

When a container is created from an image, it runs as a process on the host's kernel.

### Building blocks

1. Namespaces: like a "Process control block" but more
   1. pid
   2. net (network stack + unique ip addr)
   3. mnt (unique mount pts)
   4. ipc (unique interprocess comm)
   5. uts (hostname, domain name)
   6. user (user, group)
2. cgroups: groups for distribution of system resources in configurable manner
   1. blkio
   2. cpu
   3. cpuacct
   4. cpuset
   5. devices
   6. freezer
   7. memory
3. Union filesystem
   1. new virtual filesystem

**Container** = **instantiation** of an image.