# CS423 Fall 2023 MP0: Setting Up a Kernel Development Environment

**Assignment Due**: Sep. 12nd at 11:59 PM CT

**Last Updated**: Aug. 29th

This document will guide you through your MP0 for CS 423 Operating System Design.
It will help you prepare an environment for upcoming MPs.

Considering that you need to compile a Linux kernel all by yourself,
this MP0 may consume several minutes to a few hours of your machine time to complete.

Just keep in mind if you don't do this MP correctly, you will not be able
to work on the later MPs.

# Table of Contents
* [Overview](#overview)
* [Before your start](#before-you-start)
* [Developmental Setup](#developmental-setup)
* [Kernel Compilation](#kernel-compilation)
* [Submit Your Result](#submit-your-result)

# Overview 

### Goals

- In this MP you will learn download, compile, and install your own
	kernel.
- You will configure your development environment for upcoming projects
- You will familiarize yourself with the layout of the kernel's code 
- The kernel source code will be a helpful reference for upcoming MPs

Note: The instructions in this document are adapted from the guide at https://wiki.ubuntu.com/KernelTeam/GitKernelBuild

# Before you start

### What you need

Each student will need a Linux environment to complete all the MPs in CS423.
If you don't have a Linux bare-metal machine,
you should try to set up your own vritual machine locally, as this allows
you to have full control of your VM. In case you cannot set up your own VM,
we can help you to request an VM from Engineering IT's VM farm - but do
note that if you crash your VM with some bad kernel code, in the worst case
you will need to wait for IT to fix it, which may take a few days.

To summarize, we recommend you to use one of the following setup:

- A Linux bare-metal machine + QEMU using our QEMU script. This is
		the most recommended setup, since it allows you to easily gdb your
		kernel code.
- A Mac / Windows machine with a VM client (e.g. VMWare Fusion or
		VirtualBox). We understand that many of you do not use a Linux
		machine.

### If you want to use a Linux bare metal machine

You can go directly into the next section, yay!

### If you want to use a VM hypervisor

Depends on your current operating system, you may choose one of the following VM hypervisors if you don't have one:

**Windows**:

- **VMware**:

  VMware Workstation is a VM hypervisor developed by VMware. It has a free-for-non-commercial version called VMware Workstation Player.

  Please ensure that nested virtualization is enabled. It’s OK if you don’t enable this option, but your VM will get slower. You can enable nested virtualization using this guide: https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/17.0/com.vmware.player.win.using.doc/GUID-3140DF1F-A105-4EED-B9E8-D99B3D3F0447.html (Enable Virtualize Intel VT-x/EPT or AMD-V/RVI)

- **VirtualBox**:

  VirtualBox is a VM hypervisor developed by Oracle. It is available for free. It supports nested virtualization.

  Enabling nested virtualization for VirtualBox: https://docs.oracle.com/en/virtualization/virtualbox/6.0/admin/nested-virt.html
  
- **Windows Subsystem for Linux 2 (WSL 2)**:

  WSL 2 is a feature of the Windows operating system that enables you to run a Linux file system directly on Windows. It is available for free on Windows 10/11. An instruction on how to set it up is available at https://github.com/chinrw/wsl-qemu-kvm

- **Hyper-V**

  Hyper-V is a VM hypervisor pre-bundled with some Windows versions. It is available for free on Windows 10/11 Pro, Enterprise, and Education (it is not available on Home edition). You may refer to this guide to enable it: https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-the-hyper-v-role-through-settings

**MacOS**:

- **UTM**:

  UTM is an VM hypervisor available for free on the AppStore or its GitHub repository (https://github.com/utmapp/UTM).

  _**Note**: While creating a new VM in UTM, avoid selecting the "Apple Virtualization" option. It has been reported to lead to filesystem corruption. More details can be found here: https://github.com/utmapp/UTM/issues/4840_

- **VMware**:

  VMware Fusion is a VM hypervisor developed by VMware for MacOS. It has a free-for-non-commercial version called VMware Fusion Player. It is compatible with both Intel and Apple Silicon Macs.

- **Parallels Desktop**:

  Parallels Desktop is another popular option for MacOS users. It is a paid software (~$50 with student coupon).

**Linux**:

- **QEMU with KVM**:

  If you don't want to mess up your bare metal Linux environment, you can also install QEMU, enable KVM, and then create a VM for your MPs.

# Developmental Setup

### VM Client Setup

**If you are using a Linux bare-metal machine**:

- You don't need to perform a Client Setup. You can use your own distro.

**If you are using a VM hypervisor**:

- Install VM client with Ubuntu 22.04 as your distro. You can download the distro image at https://releases.ubuntu.com/22.04.3/ubuntu-22.04.3-desktop-amd64.iso

### Prepare for Kernel Compilation

_**Note**: the following steps assume you are running Ubuntu 22.04._

- Start configuring your machine for kernel module development by
downloading the standard development tools (**2-5 minutes**):

```bash
sudo apt-get install git bc libncurses-dev wget busybox libssl-dev libelf-dev dwarves flex bison build-essential
```

- Install QEMU (**1-3 minutes**):

```bash
# If using x86
sudo apt-get install qemu qemu-system-x86

# If using ARM
sudo apt-get install qemu qemu-system-arm
```

- Now, let's download the kernel source code (**1-5 minutes**):

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.127.tar.xz
tar -xf linux-5.15.127.tar.xz
cd linux-5.15.127
```

You can check the version of your clone kernel source via the `make
kernelversion` command. For CS423, we will use kernel version 5.15.127 (i.e. the latest LTS release at this point).

Finally, feel free to install in your VM any additional utilities or text editors (e.g., Emacs, Vim, Vscode) that you find useful.

# Kernel Compilation

### Configure a kernel

You should clone the
provided QEMU scripts and kernel config and the copy the config over:

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

- Clean the kernel source directory to prepare for compilation:

```bash
make clean
```

### Compile a kernel

We can now compile the kernel. Best practice when compiling the kernel is
to parallelize the build, one thread per available core plus one. The below
`make` command automatically detects the available CPUs on your VM to
do this (**10 minutes to 3 hours**):

```bash
make -j`nproc` LOCALVERSION=-$NETID
```

_**IMPORTANT**: CHANGE THE ABOVE LINE TO REPLACE `$NETID` WITH YOUR
NETID. The `LOCALVERSION` field is appended to the name of your
kernel._

_**Note**: Depends on how powerful your PC is, this step may take anywhere 
from several minutes to a few hours.
You may want to plug-in your laptop and put your PC into high performance mode if you want to make it faster._

Upon successful compilation, you will have new kernel image (a file called `vmlinux`) built in your kernel directory.

### Test your kernel

You can now try to boot QEMU
from your kernel source root directory with the following command (**1-2 minutes**):

```bash
../qemu-script/cs423-q
```

If everything is correct, you should have a shell inside your VM. You can
double check the kernel in the VM the command below and should see your own
kernel version (the one with your NetID):

```bash
uname -a
```

Example output:

```console
Linux CSL-PowerEdge-R750 5.15.127-tyxu #1 SMP Sun Aug 13 04:57:03 EDT 2023 x86_64 x86_64 x86_64 GNU/Linux
```

_**Note**: Depends on your environment and configuration, your output may look different.
But, please make sure the output contains `5.15.127-<netid>`._

# Submit Your Result

This is an easy one - all you need to submit is the output from the
following command:

```bash
# in your VM/QEMU
dmesg | grep 'Linux version'
```

The submission URL is: https://forms.gle/Q4TbDBt6ZrNPSLM6A

Just keep in mind if you don't do this MP correctly, you will not be able to work on the later MPs.
