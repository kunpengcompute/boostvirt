# QEMU虚拟机内存热插 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002049134326"></a>

### 简介<a name="ZH-CN_TOPIC_0000002049292604"></a>

本文主要介绍如何在使用openEuler操作系统的鲲鹏服务器上部署和使能QEMU虚拟机内存热插特性。

QEMU是一个快捷的跨平台开源计算模拟器，可以模拟许多硬件体系结构，通常用于与libvirt相结合，为用户的虚拟机提供模拟真实硬件的运行环境，方便大规模虚拟机的管理及调度。

虚拟机内存热插是一种虚拟化技术，基于QEMU实现了给处于运行状态的虚拟机动态扩展内存容量。然而，当前的ARM64架构平台上，QEMU 6.2.0的开源版本面临着一项技术局限：它不支持启动那些NUMA配置中初始内存设置为0的虚拟机，同时也无法实现这类虚拟机的内存热插功能。这一限制减少了QEMU的实际应用场景。

鲲鹏BoostKit推出的QEMU虚拟机内存热插特性基于QEMU现有的内存热插逻辑基础，通过应用Patch的方式进行了功能扩展。特性通过源码开源的方式发布在Gitee。该特性使得虚拟机的XML配置文件中可以包含一个初始内存配置为0的NUMA节点，并且允许后续通过内存热插相关命令，动态地向该NUMA节点增加内存。这一改进极大地拓展了QEMU的实际应用场景。


### 其他信息<a name="ZH-CN_TOPIC_0000002105213421"></a>

在配置特性前，请先了解虚拟机内存热插特性的基本规格、版本支持和License支持信息、使用约束与限制和应用场景。

**规格<a name="section186211624175715"></a>**

在虚拟机的XML配置文件中，当前版本最多可支持一个初始内存配置为0的NUMA节点。

**可获得性<a name="section1625164615574"></a>**

- 版本：仅支持libvirt 6.2.0和QEMU 6.2.0。
- License支持：无。

**约束与限制<a name="section3897196125818"></a>**

- 操作系统约束

    支持openEuler 22.03 LTS SP4操作系统。

- 虚拟机规格约束

    虚拟机所配置的NUMA节点中，至多可以有一个NUMA节点的初始内存配置为0。

- 热插规格约束

    为了满足QEMU的要求，虚拟机XML配置文件中必须包含**maxMemory**节点。该节点的作用是指定虚拟机能够进行热插的内存插槽数量（通过slots属性）以及通过热插所能达到的最大内存总量（通过maxMemory值）。

    >![](public_sys-resources/icon-notice.gif) **须知：** 
    >除虚拟机XML配置文件中的约束外，在不同的系统上，物理机的部分配置参数也同样会限制其上虚拟机能够进行的最大热插次数，以及通过热插能够达到的最大内存总量，具体约束项据操作系统类型而定。

**应用场景<a name="section49961711506"></a>**

在虚拟机大规模且集中管理的环境中，尤其是在云计算平台上，为了高效地管理大量虚拟机，经常需要预先在虚拟机中保留具有未分配内存的NUMA节点。这一做法旨在为后续的内存动态扩展提供便利，确保在资源需求增长时，能够迅速且有效地增加内存资源。




## 安装和使用<a name="ZH-CN_TOPIC_0000002085293185"></a>

### 环境要求<a name="ZH-CN_TOPIC_0000002085251777"></a>

本文基于openEuler操作系统提供指导，在正式操作前请确保软硬件均满足要求。

**硬件要求<a name="section26241127"></a>**

硬件要求如[**表 1** 硬件要求](#硬件要求)所示。

**表 1** 硬件要求<a id="硬件要求"></a>

|项目|说明|
|--|--|
|处理器|鲲鹏920系列处理器|


**操作系统和软件要求<a name="section153345522323"></a>**

操作系统和软件要求如[**表 2** 操作系统和软件要求](#操作系统和软件要求)所示。

**表 2** 操作系统和软件要求<a id="操作系统和软件要求"></a>

|项目|版本|获取方法|
|--|--|--|
|OS|openEuler 22.03 LTS SP4|[获取链接](https://mirrors.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP4/ISO/aarch64/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso)|
|ninja-build|1.10.2-1.oe2203sp4|在openEuler 22.03 LTS SP4系统上，确保网络畅通情况下，利用Yum工具直接安装|
|libcap-ng-devel|0.8.3-1.oe2203sp4|在openEuler 22.03 LTS SP4系统上，确保网络畅通情况下，利用Yum工具直接安装|
|libvirt|6.2.0|在openEuler 22.03 LTS SP4系统上，确保网络畅通情况下，利用Yum工具直接安装|
|QEMU|6.2.0|[获取链接](https://gitlab.com/qemu-project/qemu)|
|Add-Support-for-numa-being-initialized-with-mem-0-an.patch|-|[获取链接](https://gitee.com/kunpengcompute/qemu/pulls/1)<br>打开链接后，在页面单击“克隆/下载”，选择“下载Email Patch”即可获取补丁内容。<br><br>针对openEuler 22.03 LTS SP4所配套的QEMU 6.2.0源码而形成的Patch文件，使得QEMU支持启动那些NUMA配置中初始内存设置为0的虚拟机。<br><br>建议在“下载Email Patch”上右键，将链接另存为补丁文件，并修改命名为“Add-Support-for-numa-being-initialized-with-mem-0-an.patch”，补丁具体应用方式请参见[编译QEMU并应用特性](#编译QEMU并应用特性)。|



### 安装并验证libvirt<a name="ZH-CN_TOPIC_0000002085251765"></a>

QEMU虚拟机内存热插特性仅支持libvirt 6.2.0版本。openEuler官网提供了预编译libvirt的版本，无需编译libvirt，只需要执行本章节的操作步骤安装libvirt即可。

>![](public_sys-resources/icon-notice.gif) **须知：** 
>因特性安装过程涉及到系统文件的修改，安装过程中的各操作默认由**root**用户执行，非**root**用户下进行相关操作应自行确保具有相关权限。

1. 安装libvirt。

    ```
    yum -y install audit-libs-devel cyrus-sasl-devel fuse-devel glusterfs-api-devel glusterfs-devel gnutls-devel libacl-devel libattr-devel libcap-ng-devel libcurl-devel libnl3-devel libcap-ng-devel libnl3-devel libpciaccess-devel librados2-devel librbd1-devel libtasn1-devel libtirpc-devel libwsman-devel libxml2-devel netcf-devel numactl-devel parted-devel readline-devel scrub systemd-devel  xfsprogs-devel yajl-devel device-mapper-devel libpcap-devel libgfapi-devel libgfapi0 ninja-build python3-docutils rpcgen libxslt dnsmasq git
    yum -y install edk2* libvirt
    ```

2. 验证libvirtd服务。

    安装完成后，使用**service**命令检查系统中是否存在libvirtd服务。

    ```
    service libvirtd status
    ```

    - 回显信息中，若含有“Active: inactive \(dead\)”字样，说明系统已有libvirtd服务，但尚未启动，需要通过如下命令启动该服务。

        ```
        service libvirtd start
        ```

    - 回显信息中，若含有“Active: active \(running\)”字样，说明系统中已有libvirtd服务，且已启动，无需进行其他动作。


### 编译QEMU并应用特性<a name="ZH-CN_TOPIC_0000002049292612" id="编译QEMU并应用特性"></a>

QEMU虚拟机内存热插特性仅支持QEMU 6.2.0版本。获取QEMU 6.2.0源码后，先应用QEMU虚拟机内存热插特性Patch文件，再编译QEMU源码，即成功应用QEMU虚拟机内存热插特性。

>![](public_sys-resources/icon-notice.gif) **须知：** 
>因特性安装过程涉及到系统文件的修改，安装过程中的各操作默认由**root**用户执行，非**root**用户下进行相关操作应自行确保具有相关权限。

1. 安装编译QEMU所需的依赖包。

    ```
    yum -y install gcc gcc-c++ automake make python3 bzip2-devel zlib-devel glib2-devel pixman-devel librbd-devel openssl-devel spice*
    ```

2. 获取适配openEuler 22.03 LTS SP4版本的QEMU源码。

    ```
    git clone -b v6.2.0 --depth=1 https://git.qemu.org/git/qemu.git
    ```

3. 创建QEMU源码的分支，并将分支切换到**v6.2.0**版本。

    ```
    cd qemu
    git branch v6.2.0_patched v6.2.0
    git checkout v6.2.0_patched
    ```

4. 将QEMU源码目录与QEMU虚拟机内存热插特性Patch文件目录保持平级，并将Patch文件上传至编译环境。

    QEMU Patch的获取路径请参见[**表 2** 操作系统和软件要求](#操作系统和软件要求)。

5. 在QEMU源码目录下，将QEMU虚拟机内存热插特性Patch文件应用到QEMU源码。
    1. 检查QEMU v6.2.0原始版本的最新提交记录。

        ```
        git log -n 2
        ```

    2. 验证Patch文件Add-Support-for-numa-being-initialized-with-mem-0-an.patch是否可以成功应用到当前代码库，实际补丁文件路径可自行配置，但应确保相应补丁文件存在。

        ```
        git apply --check ../Add-Support-for-numa-being-initialized-with-mem-0-an.patch
        ```

        如果上述命令没有返回任何错误信息，表示Patch文件与当前代码库兼容，可以安全地应用。

    3. 将Patch文件应用到当前分支。

        ```
        git am ../Add-Support-for-numa-being-initialized-with-mem-0-an.patch
        ```

    4. 应用Patch文件后，再次检查最新的提交记录。

        ```
        git log -n 2
        ```

        如果Patch文件成功应用，会看到最新的提交记录已经更新。

        >![](public_sys-resources/icon-note.gif) **说明：** 
        >如果使用的是**git apply**命令来应用Patch文件，那么它不会创建一个新的提交记录，此时如果没有返回错误回显，即表示Patch文件已成功应用。

        一旦成功应用Patch文件，当前目录下的QEMU源码就已经包含了所需的QEMU虚拟机内存热插特性，可以准备编译QEMU。

6. 编译QEMU。

    ```
    rm -rf build && mkdir build
    cd build/
    ../configure --disable-werror --enable-spice --enable-spice-protocol  --target-list=aarch64-softmmu --cc="gcc" --extra-cflags="-Wno-error" --disable-docs --enable-virtfs --enable-numa
    make -j 64
    ```

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >**make -j 64**命令中的**64**表示同时执行64个线程。用户可以根据系统的CPU核数或可用资源来调整这个数值。

7. 在build目录下安装并部署QEMU。

    ```
    make install
    ```

    部署QEMU完成后，环境中已经成功应用QEMU虚拟机内存热插特性。合入特性后，默认特性为使能状态，无需手动开启。

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >利用XML文件定义虚拟机时，需要在该XML文件的 **<devices\>** 节点下新增 **<emulator\>** 节点，并将其路径指向刚刚安装的QEMU可执行文件。例如，在本例中，应新增如下内容。
    >```
    ><emulator>/usr/local/bin/qemu-system-aarch64</emulator>
    >```




