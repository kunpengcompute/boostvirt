# QEMU VM Memory Hotplug Feature Guide

## Feature Description<a name="EN-US_TOPIC_0000002049134326"></a>

### Introduction<a name="EN-US_TOPIC_0000002049292604"></a>

This document describes how to deploy and enable the QEMU VM memory hotplug feature on a Kunpeng server running the openEuler OS.

QEMU is a cross-platform computing emulator that is quick and open-source. It can emulate a variety of hardware architectures and usually works together with libvirt to provide user VMs with emulated hardware operating environment, offering easy management and scheduling of large-scale VMs.

VM memory hotplug is a virtualization technology that dynamically expands the memory capacity of a running VM based on QEMU. However, on the current AArch64 architecture platform, the open-source QEMU 6.2.0 faces a technical limitation: It cannot start VMs whose initial memory in the NUMA configuration is configured to 0, and cannot provide the memory hotplug function to such VMs. This limitation narrows the QEMU application scenarios.

The Kunpeng BoostKit QEMU VM memory hotplug feature is based on QEMU's existing memory hotplug logic and extends functions through patches. The feature is open-sourced and released on Gitee. This feature makes the VM's XML configuration file contain a NUMA node with 0 initial memory and dynamically adds memory to the NUMA node using memory hotplug commands. This improvement greatly widens the QEMU application scenarios.

### Other Information<a name="EN-US_TOPIC_0000002105213421"></a>

Before configuring the VM memory hotplug feature, learn the specifications, supported version, license requirement, constraints, and application scenarios of this feature.

**Specifications<a name="section186211624175715"></a>**

In the XML configuration file of a VM, a maximum of one NUMA node whose initial memory is 0 is supported.

**Availability<a name="section1625164615574"></a>**

- Supported version: Only libvirt 6.2.0 and QEMU 6.2.0 are supported.
- License: none.

**Constraints<a name="section3897196125818"></a>**

- OS

    openEuler 22.03 LTS SP4 is supported.

- VM specification

    Among the NUMA nodes configured for a VM, a maximum of one NUMA node whose initial memory is 0 is supported.

- Hotplug specification

    The VM XML configuration file must contain the `maxMemory` node as required by QEMU. This node specifies the number of memory slots (setting the `slots` attribute) dedicated for the VM's memory hotplug and the maximum total memory that can be achieved through hotplug (setting the value of `maxMemory`).

    >![](public_sys-resources/icon-notice.gif) **NOTICE:**
    >In addition to the constraints posed by the VM XML configuration file, some configuration parameters of the physical machine also put constraints on the maximum number of hotplug operations that can be performed on the VM and the maximum memory capacity that can be reached through hotplug. The specific constraints depend on the OS type.

**Application Scenarios<a name="section49961711506"></a>**

In environments where a large number of VMs are managed in a centralized manner, especially on cloud computing platforms, NUMA nodes that are temporarily allocated with 0 memory are often reserved in VMs in advance, aiming to efficiently manage large-scale VMs. This method ensures that memory resources can be quickly increased and allocated when more memory is required, facilitating dynamic memory expansion.

## Installation and Usage<a name="EN-US_TOPIC_0000002085293185"></a>

### Environment Requirements<a name="EN-US_TOPIC_0000002085251777"></a>

This document provides guidance based on the openEuler OS. Before performing operations, ensure that your hardware and software meet the requirements.

**Hardware Requirements<a name="section26241127"></a>**

[**Table 1**](#hardware-requirement) lists the hardware requirement.

**Table 1** Hardware requirement<a id="hardware-requirement"></a>

|Item|Description|
|--|--|
|Processor|Kunpeng 920 series|

**OS and Software Requirements<a name="section153345522323"></a>**

[**Table 2**](#os-and-software-requirements) lists the OS and software requirements.

**Table 2** OS and software requirements<a id="os-and-software-requirements"></a>

|Item|Version|How to Obtain|
|--|--|--|
|OS|openEuler 22.03 LTS SP4|[Link](https://mirrors.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP4/ISO/aarch64/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso)|
|ninja-build|1.10.2-1.oe2203sp4|Install it using Yum on openEuler 22.03 LTS SP4 when the network connection is normal.|
|libcap-ng-devel|0.8.3-1.oe2203sp4|Install it using Yum on openEuler 22.03 LTS SP4 when the network connection is normal.|
|libvirt|6.2.0|Install it using Yum on openEuler 22.03 LTS SP4 when the network connection is normal.|
|QEMU|6.2.0|[Link](https://gitlab.com/qemu-project/qemu)|
|Add-Support-for-numa-being-initialized-with-mem-0-an.patch|-|[Link](https://gitee.com/kunpengcompute/qemu/pulls/1)|

### libvirt Installation and Verification<a name="EN-US_TOPIC_0000002085251765"></a>

The QEMU VM memory hotplug feature only supports libvirt 6.2.0. The openEuler official website provides a precompiled libvirt version that does not require your manual compilation. You only need to perform the operations in this section to install libvirt.

>![](public_sys-resources/icon-notice.gif) **NOTICE:**
>The feature installation involves system file modification. By default, all operations during the installation are performed by the `root` user. If you are a non-`root` user, ensure that you have corresponding permissions.

1. Install libvirt.

    ```shell
    yum -y install audit-libs-devel cyrus-sasl-devel fuse-devel glusterfs-api-devel glusterfs-devel gnutls-devel libacl-devel libattr-devel libcap-ng-devel libcurl-devel libnl3-devel libcap-ng-devel libnl3-devel libpciaccess-devel librados2-devel librbd1-devel libtasn1-devel libtirpc-devel libwsman-devel libxml2-devel netcf-devel numactl-devel parted-devel readline-devel scrub systemd-devel  xfsprogs-devel yajl-devel device-mapper-devel libpcap-devel libgfapi-devel libgfapi0 ninja-build python3-docutils rpcgen libxslt dnsmasq git
    yum -y install edk2* libvirt
    ```

2. Verify the libvirtd service.

    After the installation, run the `service` command to check whether the libvirtd service exists in the system.

    ```shell
    service libvirtd status
    ```

    - If "Active: inactive (dead)" is returned, the libvirtd service (not started) exists in the system. Run the following command to start the service:

        ```shell
        service libvirtd start
        ```

    - If "Active: active (running)" is returned, the libvirtd service is running in the system.

### QEMU Compilation and Feature Application<a name="EN-US_TOPIC_0000002049292612" id="qemu-compilation-and-feature-application"></a>

The QEMU VM memory hotplug feature only supports QEMU 6.2.0. Obtain the QEMU 6.2.0 source code, apply the patch file for QEMU VM memory hotplug, and compile the QEMU source code. The memory hotplug feature is successfully applied.

>![](public_sys-resources/icon-notice.gif) **NOTICE:**
>The feature installation involves system file modification. By default, all operations during the installation are performed by the `root` user. If you are a non-`root` user, ensure that you have corresponding permissions.

1. Compile and install the dependency packages required by QEMU.

    ```shell
    yum -y install gcc gcc-c++ automake make python3 bzip2-devel zlib-devel glib2-devel pixman-devel librbd-devel openssl-devel spice*
    ```

2. Obtain the QEMU source code that matches openEuler 22.03 LTS SP4.

    ```shell
    git clone -b v6.2.0 --depth=1 https://git.qemu.org/git/qemu.git
    ```

3. Create a branch of the QEMU source code and switch the branch to the `v6.2.0` version.

    ```shell
    cd qemu
    git branch v6.2.0_patched v6.2.0
    git checkout v6.2.0_patched
    ```

4. Ensure that the QEMU source code directory is at the same level as the patch file (for enabling the feature) directory, and upload the patch file to the compilation environment.

    For details about how to obtain the patch file, see [**Table 2**](#os-and-software-requirements).

5. In the QEMU source code directory, apply the patch file to the QEMU source code.
    1. Check the latest commit record of the original QEMU v6.2.0.

        ```shell
        git log -n 2
        ```

    2. Check whether the patch file `Add-Support-for-numa-being-initialized-with-mem-0-an.patch` can be successfully applied to the current code library. Configure the actual patch file path as required and ensure that the corresponding patch file exists.

        ```shell
        git apply --check ../Add-Support-for-numa-being-initialized-with-mem-0-an.patch
        ```

        If no error is returned, the patch file is compatible with the current code library for normal application.

    3. Apply the patch file to the current branch.

        ```shell
        git am ../Add-Support-for-numa-being-initialized-with-mem-0-an.patch
        ```

    4. After applying the patch file, check the latest commit record again.

        ```shell
        git log -n 2
        ```

        If the latest commit record is updated, the patch file is successfully applied.

        >![](public_sys-resources/icon-note.gif) **NOTE:**
        >If `git apply` is used to apply the patch file, no new commit record is created. If no error message is returned, the patch file is successfully applied.

        Once the patch file is successfully applied, the QEMU source code in the current directory already contains the QEMU VM memory hotplug feature. Prepare to compile QEMU.

6. Compile QEMU.

    ```shell
    rm -rf build && mkdir build
    cd build/
    ../configure --disable-werror --enable-spice --enable-spice-protocol  --target-list=aarch64-softmmu --cc="gcc" --extra-cflags="-Wno-error" --disable-docs --enable-virtfs --enable-numa
    make -j 64
    ```

    >![](public_sys-resources/icon-note.gif) **NOTE:**
    >In the preceding command, `make -j 64` indicates that 64 threads are executed at the same time. You can adjust this value based on the number of CPU cores or available resources.

7. Install and deploy QEMU in the `build` directory.

    ```shell
    make install
    ```

    After QEMU is deployed, the QEMU VM memory hotplug feature has been successfully applied in the environment. After the feature is applied, it is enabled by default.

    >![](public_sys-resources/icon-note.gif) **NOTE:**
    >When using an XML file to define a VM, you need to add the `<emulator>` node under the `<devices>` node in the XML file and point the path of `<emulator>` to the QEMU executable file installed just now. In this example, add the following content:
>
    >```xml
    ><emulator>/usr/local/bin/qemu-system-aarch64</emulator>
    >```
