---
layout: post
title:  KVM Virtualization Technology
date:   2021-05-06
categories:
  - Cloud Computing
tags:
  - KVM
  - Learning notes
---

Introduction to KVM Virtualization Technology

Content

{% include toc %}

# Chapter 1 Virtualization and Cloud Computing

## 1.1 Cloud computing concept

SaaS (Software as a Service)

PaaS (Platform as a Service)

IaaS (Infrastructure as a Service)



## 1.2 Cloud computing technology

### 1.2.1 Map/Reduce

It is a programming model developed by Google. It is a simplified distributed programming model and an efficient task scheduling model for parallel computing of large-scale data sets. It is used in distributed search, distributed sorting, machine learning, machine translation, etc., including Internet web search indexes such as Google.

Library implementation: Hadoop under JAVA language

## 1.3 Virtualization technology

### 1.3.1 Software virtualization and hardware virtualization

Key step: The virtualization layer must be able to intercept the direct access of computing elements to physical resources and redirect them to the virtual resource pool.

1. Software solution

Pure software virtualization, using pure software methods to intercept and simulate access to physical platforms on existing physical platforms.

QEMU, through pure software to simulate the instruction fetching, decoding and execution of X86 platform processors, the client's instructions are not directly executed on the physical platform. All instructions are simulated by software, so the performance is often poor, but virtual machines of different architecture platforms can be simulated on the same platform.

2. Hardware solution

Hardware virtualization, the physical platform itself provides hardware support for intercepting and redirecting special instructions, and even new hardware will provide additional resources to help software virtualize key hardware resources, thereby improving performance.

Inter VT virtualization technology: Pentium, Core, Xeon, Itanium

## 1.4 Introduction to KVM

KVM stands for Kernel Virtual Machine, developed by the Israeli company Qumranet. KVM did not write a new hypervisor from the bottom up, but chose to turn the Linux kernel itself into a hypervisor by inserting a new module based on the Linux kernel. In October 2006, the source code of the KVM module was officially accepted into the Linux kernel. Xen is different, it will replace the kernel and manage system resources by itself.

In the KVM architecture, virtual machines are implemented as regular Linux processes and scheduled by the standard Linux scheduler. KVM itself does not perform any simulation, and requires a user space program (QEMU) to set up an address space of a client virtual server through the /dev/kvm interface, provide it with simulated I/O, and map its video display back to the host's display.

![image-20210429174649641](https://user-images.githubusercontent.com/48710834/117282972-e8905580-ae97-11eb-9548-fa393d5107c5.png)

## 1.5 Introduction to Xen

Xen is a virtual machine management program that runs directly on system hardware. Xen inserts a virtualization layer between the system hardware and virtual machines, converting the system hardware into a logical computing resource pool, in which Xen can dynamically allocate resources to any operating system or application.

![image-20210429174442599](https://user-images.githubusercontent.com/48710834/117283025-f80f9e80-ae97-11eb-9445-5255892368be.png)

Xen is designed as a microkernel implementation, which is only responsible for managing processor and memory resources.

# Chapter 2 Introduction to KVM Principles

## 2.1 Introduction to Linux Operating System

Modularly designed Linux:

Operating system kernel design has always been divided into two camps: microkernel and single kernel;

Single kernel: The entire kernel is implemented as a single large process as a whole, and runs in a single address space at the same time. All kernel services run in such a large kernel space, and communication between kernels can be simply implemented as function calls.

Microkernel: It is not implemented as a single large process. Instead, the kernel's functions are divided into multiple independent processes. Each process is called a server. Multiple weapon programs run in their own address space. Only a small number of core servers run in privileged mode. The communication between servers uses the inter-process communication mechanism.

Linux adopts a pragmatic design. The Linux kernel is designed as a single kernel, while supporting modular design and the ability to dynamically load kernel modules. In addition to core kernel functions such as process switching and memory management, most kernel functions are designed and implemented as separate kernel modules. During the operation of the kernel, according to demand, it is dynamically loaded and connected to the kernel space for operation. Unused modules can also be dynamically unloaded during the operation.

## 2.2 Virtualization model

![image-20210429180256129](https://user-images.githubusercontent.com/48710834/117283106-0cec3200-ae98-11eb-9816-8d8a46b89393.png)

Hypervisor can also be called VMM (virtual machine monitor), which runs on the physical system, manages the real physical hardware platform, and provides a corresponding virtual hardware platform for each virtual client.

## 2.3 KVM architecture

The basic architecture of virtual machines is divided into two types:

Type 1: After the system is powered on, the virtual machine monitor is first loaded and run, and the traditional operating system runs in the virtual machine it creates. In a sense, it can be regarded as the operating system kernel. Typical examples are Xen, VMware's ESX/ESXi and Microsoft's Hyper-V.

Type 2: After the system is powered on, the general operating system is still running. The virtual machine monitor, as a special application, can be regarded as an extension of the operating system function. Typical examples are KVM, VMware Workstation, and VirtualBox.

![image-20210429181440975](https://user-images.githubusercontent.com/48710834/117283058-01990680-ae98-11eb-8fe6-aaa7932f7d01.png)

KVM itself does not perform any device simulation. It is loaded into the kernel space on demand during runtime.

## 2.4 KVM module

The KVM module is the core part of the KVM virtual machine. Its main function is to initialize the CPU hardware, turn on the virtualization mode, and then run the virtual client in the virtual machine mode, and provide certain support for the operation of the virtual client.

The communication interface between the KVM module and the user space QEMU is mainly a series of IOCTL calls for special device files.

Creating a virtual machine: The most important IOCTL call for the /dev/kvm file can be understood as KVM creating the corresponding kernel data structure for a specific virtual client. At the same time, KVM will also return a file handle to represent the created virtual machine. Then you can create a mapping relationship between the user space virtual address and the client physical address and the real memory physical address, etc.

Executing the virtual processor: The virtual machine prepared by the user space is placed in the non-root mode in the virtualization mode with the support of the KVM module and starts to execute binary instructions.

Memory virtualization: The virtual address of the client program is converted into the client physical address and then into the real physical address. KVM uses shadow page tables to solve this problem. The shadow page table is complex to implement and sometimes has a large overhead. A secondary page table can also be introduced, which is called a two-dimensional paging mechanism.

## 2.5 QEMU device model

The QEMU virtual machine is a pure software implementation with low performance, but its advantage is that the function of the virtual machine can be realized on a platform that supports the compilation and operation of QEMU itself, and the virtual machine can be on a different architecture from the host machine. The QEMU code contains a complete set of virtual machine implementations, including processor virtualization, memory virtualization, and virtual device simulation used by KVM.

During the operation of the virtual machine, QEMU will enter the kernel through the system call provided by the KVM module. The KVM module is responsible for putting the virtual machine into a special mode of the processor. When the virtual machine performs input and output operations, the KVM module will return to QEMU from the last system call exit, and QEMU will be responsible for parsing and simulating these devices. From the perspective of QEMU, it can also be said that QEMU uses the virtualization function of the KVM module to provide hardware virtualization acceleration for its own virtual machines, thereby greatly improving the performance of the virtual machine.

# Chapter 3 Building a KVM environment