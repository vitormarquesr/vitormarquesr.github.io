---
title: Creating an environment for kernel development
categories: [Open Source Software Development, Kernel Contribuitions, ]
tags: [linux, kernal, refactoring, patch, iio, open-source, kernel-linux]
render_with_liquid: false
---
Collaborators [Gabriel Lima](https://gabriellimmaa.github.io/), [Gabriel José](https://gabrielpereir4.github.io/gabriel-portfolio/), [Vitor Marques](https://vitormarquesr.github.io/blog/).

As part of my journey into Linux kernel development, I followed a series of four hands-on tutorials designed to provide a practical introduction to building, configuring, and testing custom kernels. These tutorials focused on using virtualization tools like QEMU and libvirt, automating workflows with KW, and developing kernel modules and drivers. This blog post summarizes my experience with each tutorial, the challenges faced, and the key lessons learned throughout the process.

## Tutorial 1 — Setting up a test environment for Linux Kernel Dev using QEMU and libvirt (26/02)

The goal of this first tutorial was to set up a test environment for Linux kernel development using QEMU and libvirt. The idea was to create a virtual machine capable of compiling and running custom kernels safely.

[Link - Tutorial 1](https://flusp.ime.usp.br/kernel/qemu-libvirt-setup/)

## Tutorial 2 — Building and booting a custom Linux kernel for ARM using KW (12/03)

In this second tutorial, the focus was on compiling the Linux kernel for the ARM architecture and testing it in a virtual machine. Unlike the original tutorial, I used **KW (Kernel Workflow Tool)** to automate part of the process, which greatly simplified the configuration and build steps.

I faced some issues while mounting the file system inside the VM due to partitioning, but with guidance from the instructor, we were able to resolve it.

[Link - Tutorial 2](https://flusp.ime.usp.br/kernel/build-linux-for-arm/)

## Tutorial 3 — Introduction to Linux kernel build configuration and modules (19/03)

The third tutorial introduced kernel module development and basic kernel build configurations. The activity consisted of creating a simple module, compiling it, and loading it into the kernel for testing.

This step went smoothly. I followed all the instructions in the guide and was able to complete the tutorial without errors or setbacks.

[Link - Tutorial 3](https://flusp.ime.usp.br/kernel/modules-intro/)

## Tutorial 4 — Introduction to Linux kernel Character Device Drivers (26/03)

The final tutorial covered the basics of character device drivers in Linux. We created a simple driver to understand how the kernel interacts with this type of device.

The content was clear and well-structured, and I was able to implement the proposed driver without any issues. It was a good hands-on introduction to kernel driver development.

[Link - Tutorial 4](https://flusp.ime.usp.br/kernel/char-drivers-intro/)
