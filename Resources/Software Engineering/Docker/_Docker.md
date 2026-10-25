# Docker

Docker is a set of PaaS products. It uses OS-level virtualizations to deliver software in package called **containers**.

# Containers vs VMs

**Virtualization** is the process in which a system singular resource like RAM, CPU, Disk, or Networking can be ‘virtualized’ and represented as multiple resources.

Key difference between containers and virtual machines:
virtual machines virtualize an entire machine down to the hardware layers and containers only virtualize software layers above the operating system level.

![SWTM-2060_Diagram_Containers_VirtualMachines_v03.png](Docker/SWTM-2060_Diagram_Containers_VirtualMachines_v03.png)

## Container

Containers are lightweight software packages (notice how we don’t call it an OS here) that contain all the dependencies required to execute the contained software application. 

The dependencies included in a container exist in stack levels that are higher than the operating system.

### **Pros**

- **Iteration speed:** Containers are lightweight, contain only high level software. Very fast to iterate on and modify.
- **Robust ecosystem:** Huge public repo of pre-made containers available for reuse.

### **Cons**

- **Shared host exploits:** Containers all share same hardware system below OS; it is possible that an exploit in one container could break out of the container and affect the shared hardware.

## Virtual Machine

Virtual machines are heavy software packages that provide complete emulation of low level hardware devices like CPU, Disk and Networking devices. 

They may also include a complementary software stack to run on the emulated hardware (the OS and such).

These hardware and software packages combined produce a fully functional snapshot of a computational system.

### **Pros**

- **Full isolation security:** Virtual machines run in isolation as a fully standalone system. One exploitable VM doesn’t compromise the other VMs on the same host.
- **Interactive development:** Containers are usually static definitions of the expected dependencies and configuration needed to run the container. 
Virtual machines are more dynamic and can be interactively developed. Once the basic hardware definition is specified for a virtual machine the virtual machine can then be treated as a bare bones computer.

### **Cons**

- **Iteration speed:** VMs are time consuming to build and regenerate because they encompass a full stack system. It also takes more time to test and validate a change.
- **Storage size cost:** VMs can take up a lot of storage space on host machines.

## Linux Container (LXC)

Basically a lightweight virtual machine, but instead of installing a full blown operating system on each VM, it will only install everything that is strictly necessary.

## Docker

Unlike how a CM would work, Docker would not include the functionality of an OS at all.

It uses a lot of resources of the host OS. It does that by using the Linux Kernel’s functionalities.
Multiple Docker containers running on the same machine will share the CPU’s cycles and controls.

[_LXC vs Docker](_LXC%20vs%20Docker.md)