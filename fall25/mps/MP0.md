# CS 423 Fall 2025 MP0: Setting Up a Kernel Development Environment

**Assignment Due**: See the [homepage](https://cs423-uiuc.github.io/fall25/)

**Last Updated**: Aug. 28th, 2025

This document will guide you through your MP0 for CS 423 Operating System
Design. It will help you prepare an environment for upcoming MPs.

Considering that you need to compile a Linux kernel all by yourself, this
MP0 may consume several minutes to a few hours of your machine time to
complete.

> **If you don't do this MP correctly, you will not be able to work on the
  later MPs.**

# Table of Contents
* [Overview](#overview)
* [Before You Start](#before-you-start)
* [Environmental Setup](#environmental-setup)
* [Kernel Compilation](#kernel-compilation)
* [Submit Your Result](#submit-your-result)

# Overview

### Goals

- In this MP you will learn to download, compile, and test your own kernel.
- You will configure your development environment for upcoming MPs.
- The kernel source code will be a helpful reference.

Note: The instructions in this document are adapted from the guide at
https://wiki.ubuntu.com/KernelTeam/GitKernelBuild

# Before You Start

### What you need

Each student will need a Linux environment to complete all the MPs in CS 423.
Specifically, you will need a Linux environment to compile your own kernel.
We will then provide a QEMU script, and you will use QEMU to test your kernel
and all the upcoming MPs.

We understand that many of you do not use a Linux machine, or don't want to
mess up your bare metal Linux environment. In this case, you should try to set
up a Linux VM locally, as this allows you to have full control of your VM. In
case you cannot set up your own VM, we can help you to request an VM from
Engineering IT's VM farm, but do note that if you crash your VM, in the worst
case you will need to wait for IT to fix it, which may take a few days and
might be devastating.

To summarize, We recommend you use one of the following setups:

- A Linux bare-metal machine with QEMU using our QEMU script.
- A Linux VM with QEMU using our QEMU script.

You are free to use your favorite distro, but please note that the following
sections of this document assumes you are using Ubuntu 24.04.

# Environmental Setup

### Linux VM setup

> If you are planning to use a Linux bare-metal machine, please go directly to
  the next section.

Before setting up a Linux VM, you will need to install a VM hypervisor.
Depends on your operating system, there are some recommendations if you don't
have one.

We will also introduce how to enable nested virtualization for your hypervisor.
Your kernel runs in a QEMU VM inside your Linux VM, which is a case of nested
virtualization. Without nested virtualization, QEMU will use software. As a
result, your kernel will run much slower.

**Windows**:

Modern Windows PCs often ships with a Hyper-V hypervisor, which can interfere
all other third-party VM hypervisors, to support various security features
like VBS and HVCI. In this case, we recommend WSL 2 only. **However, keep in
mind that if you are using Windows 10, you won't be able to enable nested
virtualization in your WSL 2 VM.**

Checking the status or disabling the Hyper-V hypervisor is actually quite
complicated. If you are interested, you can follow this guide:
https://community.broadcom.com/vmware-cloud-foundation/discussion/how-to-disable-hyper-v-in-windows-11-24h2 (do not follow Step 6 unless you
are using Win11 24H2). In the case that you are using Windows 10 or you
want to use third-party VM hypervisors, you may want to disable it.
However, you are warned that you will lose access to your current WSL 2 VM
and various security features.

- **Windows Subsystem for Linux 2 (WSL 2) on Hyper-V**:

  WSL 2 is a feature of the Windows operating system that enables you to run
  Linux directly on Windows. It is available for free on Windows 10/11. An
  instruction on how to set it up is available at
  https://learn.microsoft.com/en-us/windows/wsl/install.

  Then, run the following command in your WSL 2 shell:

  ```bash
  sudo mkdir /lib/modules
  ```

  If you are running Windows 11, you can enable nested virtualization using
  this guide: https://github.com/chinrw/wsl-qemu-kvm.
  
  > After setting it up, you can go directly to the next section.

- **VirtualBox**
  
  VirtualBox is a VM hypervisor developed by Oracle. You can download it at
  https://www.virtualbox.org/.
  
  You can enable nested virtualization using this guide:
  https://www.virtualbox.org/manual/topics/AdvancedTopics.html#nested-virt.

**macOS**:

If you are using ARM-based Mac, we recommend UTM.

- **UTM**:

  UTM is a VM hypervisor available for free on Homebrew or its GitHub
  repository at https://github.com/utmapp/UTM.

  Please ensure that you select "Use Apple Virtualization" while creating a
  new VM in UTM. This can enable nested virtualization if your machine
  supports it.

- **VirtualBox**
  
  VirtualBox is a VM hypervisor developed by Oracle. You can download it at
  https://www.virtualbox.org/.
  
  You can enable nested virtualization using this guide:
  https://www.virtualbox.org/manual/topics/AdvancedTopics.html#nested-virt.

**Linux**:

- **QEMU with KVM**:

  If you don't want to mess up your bare metal Linux environment, you can also
  install QEMU, enable KVM, and then create a VM for your MPs.

Then, create a virtual machine and install Ubuntu Server 24.04. You can download
the distro image at:

  - x86: https://ubuntu.com/download/server
  - ARM: https://ubuntu.com/download/server/arm
  
Ubuntu Server does not come with a GUI. We recommend you check the "Install
OpenSSH Server" option during the installation and connect to your VM using
SSH for development. Most modern operating systems ships with a SSH client,
and you can use your favorite terminal emulator (like Windows Terminal) to
connect to your VM.

### Prepare for kernel compilation

> The following steps assume you are running Ubuntu 24.04.

Start configuring your machine for kernel module development by downloading
the standard development tools (**2-5 minutes**):

```bash
sudo apt-get update
sudo apt-get install git bc libncurses-dev wget busybox libssl-dev libelf-dev dwarves flex bison build-essential python3
```

Install QEMU (**1-3 minutes**):

```bash
# If using x86
sudo apt-get install qemu-system-x86

# If using ARM
sudo apt-get install qemu-system-arm
```

Now, let's download the kernel source code (**1-5 minutes**):

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.165.tar.xz
tar -xf linux-5.15.165.tar.xz
cd linux-5.15.165
```

You can check the version of your clone kernel source via the
`make kernelversion` command. For CS 423, we will use kernel version
5.15.165.

Finally, feel free to install in your VM any additional utilities or text editors
(e.g., Emacs, Vim) that you find useful. You can also use external editors with
SSH for development (like VSCode).

# Kernel Compilation

### Configure your kernel

You should clone the provided QEMU scripts and kernel config and the copy
the config over:

```bash
git clone https://github.com/cs423-uiuc/qemu-script.git ../qemu-script

# If using x86
cp ../qemu-script/.config-x86-64 .config

# If using ARM
cp ../qemu-script/.config-arm64 .config
```

To apply the kernel config, do:

```bash
make olddefconfig
```

This will create a config for your kernel based on the old .config file. In
case there are any new config without values, it will use the default
automatically.

Clean the kernel source directory to prepare for compilation:

```bash
make clean
```

### Compile your kernel

We can now compile the kernel. Best practice when compiling the kernel is
to parallelize the build, one thread per available core plus one. The below
`make` command automatically detects the available CPUs on your VM to do
this (**10 minutes to 3 hours**):

```bash
make -j`nproc` LOCALVERSION=-$NETID
```

> Change the above line to replace `$NETID` with your NetID.
  `$` denotes environment variables in shell and should also be removed
  The `LOCALVERSION` field will be appended to the name of your kernel.

> Depends on how powerful your PC is, this step may take anywhere from
  several minutes to a few hours. You may want to plug-in your laptop and
  put your PC into high performance mode if you want to make it faster.

> If you are using your own distro with gcc >= 15, the default `-std=c23`
  option will break the build. It's recommended to build in docker environment
  then copy the directory to host environment.

Upon successful compilation, you will have new kernel image (a file called
`vmlinux`) built in your kernel directory.

### Test your kernel

You can now try to boot QEMU from your kernel source root directory with
the following command (**1-2 minutes**):

```bash
../qemu-script/cs423-q
```

> If QEMU complains about KVM and refuses to start, that means you are
  having issues on KVM/nested virtualization. Make sure you follow the
  guide. You can also use `-t` flag to force software virtualization.

If everything is correct, you should have a shell inside your VM. You can
double check the kernel in the VM the command below and should see your own
kernel version (the one with your NetID):

```bash
uname -a
```

Example output:

```console
Linux CSL-PowerEdge-R750 5.15.165-tyxu #1 SMP Sun Aug 13 04:57:03 EDT 2023 x86_64 x86_64 x86_64 GNU/Linux
```

> Depends on your environment and configuration, your output may look
  different. But, please make sure the output contains `5.15.165-<netid>`.

To quit the QEMU and halt your kernel, you can simply exit the shell by
typing `exit`. But just in case that you break your kernel, press Ctrl-A,
and then press X. This will force QEMU to quit.

# Submit Your Result

This is an easy one - you just need to submit to this **[form](https://docs.google.com/forms/d/e/1FAIpQLScGT0yraO3Y4nfpJaF033hxz8F4ZNoMkycOj4qbsYJQNYsPIw/viewform?usp=dialog)**
with the output from the following command:

```bash
# in your VM/QEMU
dmesg | grep 'Linux version'
```
