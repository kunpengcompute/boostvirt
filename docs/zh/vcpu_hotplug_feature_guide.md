# vCPU热插拔 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002518527652"></a>

### 简介<a name="ZH-CN_TOPIC_0000002550127409"></a>

本文主要介绍如何在使用openEuler操作系统的鲲鹏服务器上部署并使能vCPU热插拔特性。

vCPU热插拔技术可以在虚拟机运行状态下增加或减少vCPU数量，实现不中断业务条件下动态调整vCPU资源。

现代操作系统要尽可能长期可靠地运行而不下线，并且具备足够强大的容错机制，即RAS（Reliability, Availability and Serviceability）。其中，Availability（可用性）要求系统出现一些小的问题也不会影响整个系统的正常运行，在某些情况下可以进行Hot Plug的操作，替换有问题的组件，从而严格确保系统的downtime时间在一定范围内。在虚拟化场景下，除了RAS外，还要求资源具备动态调度的能力。客户在创建虚拟机的时候不能够准确预知虚拟机中的业务对设备资源的压力情况。如果虚拟机中某设备压力长期偏高或者偏低，就会制约业务的运行速度或者造成资源浪费。通过vCPU热插拔，可以有效解决此类问题。

资源弹性是云计算的核心优势之一，而vCPU热插拔是实现CPU算力弹性的关键技术之一。vCPU热插拔的价值不限于：

- 加快虚拟机启动速度。特别对于轻量化场景的受益较大。比如Kata安全容器初始只配置1个vCPU，等启动完成后，可以热插更多vCPU。
- 按需使用资源，从而优化业务成本。开发者根据业务负载需求，在线调整虚拟机vCPU数量。负载大时增加资源，负载小时减少资源。

### 约束与限制<a name="ZH-CN_TOPIC_0000002550127413"></a>

- 如果处理器为AArch64架构，创建虚拟机时指定的虚拟机芯片组类型（machine）需为virt-4.2及以上版本。如果处理器为x86\_64架构，创建虚拟机时指定的虚拟机芯片组类型（machine）需为pc-i440fx-1.5及以上版本。
- 在配置Guest NUMA的场景中，必须把属于同一个Socket的vCPU配置在同一vNode中，否则热插vCPU后可能导致虚拟机软锁死，进而可能导致虚拟机无法继续正常运行。
- 虚拟机在迁移、休眠唤醒、快照过程中均不支持vCPU热插。
- 虚拟机vCPU热插是否自动上线取决于虚拟机操作系统自身逻辑，虚拟化层不保证热插vCPU自动上线。
- 虚拟机启动、关闭、重启过程中可能出现热插vCPU失效的情况，但再次重启会生效。
- 热插虚拟机vCPU的时候，如果新增vCPU数目不是虚拟机vCPU拓扑配置项中Cores的整数倍，可能会导致虚拟机内部看到的vCPU拓扑是混乱的，建议每次新增的vCPU数目为Cores的整数倍。
- 特性增强：老版本（例如，openEuler-22.03-SP4）已具备vCPU热插能力，openEuler 24.03 LTS版本新增了对 vCPU热拔的支持，适应更多使用场景。相对老版本，热插拔协议发生了一些变化，新老版本的热插拔不能兼容运行。即Guest内核版本和主机侧QEMU版本需配套才能实现热插拔功能。
- 热插拔规格约束
    - 为了满足QEMU的要求，虚拟机XML配置文件中必须包含**vcpu**节点。该节点的作用是指定虚拟机初次启动时的vCPU个数，以及能够通过vCPU热插所能达到的最大vCPU个数。
    - vCPU热插同时受限于Hypervisor和GuestOS支持的最大CPU数目，具体约束项根据操作系统类型而定。

### 应用场景<a name="ZH-CN_TOPIC_0000002550007407"></a>

- 基于vCPU热插拔机制加快虚拟机启动速度。

    特别对于轻量安全容器场景受益较大。比如Kata安全容器初始只配置1个vCPU，等启动完成后热插更多vCPU。

- 云厂商基于vCPU热插拔为用户按需扩展，用户根据业务负载需求，申请在线调整虚拟机vCPU数量，不影响业务运行。

## 特性使用<a name="ZH-CN_TOPIC_0000002518687568"></a>

### 环境要求<a name="ZH-CN_TOPIC_0000002518527648"></a>

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
|OS|openEuler 24.03 LTS|[获取链接](https://mirrors.pku.edu.cn/openeuler/openEuler-24.03-LTS/ISO/aarch64/openEuler-24.03-LTS-everything-aarch64-dvd.iso)|
|libvirt|9.10.0|使用Yum工具直接安装。|
|QEMU|8.2.0|使用Yum工具直接安装。|

### 特性使能<a name="ZH-CN_TOPIC_0000002550127411"></a>

使用vCPU热插功能，需要在创建虚拟机时配置虚拟机当前的CPU数目、虚拟机所支持的最大CPU数目，以及虚拟机芯片组类型（对于AArch64架构，需为virt-4.2及以上版本。对于x86\_64架构，需为pc-i440fx-1.5及以上版本）。这里以AArch64架构虚拟机为例，配置模板如下。

```xml
<domain type='kvm'>
...
<vcpu placement='static' current='m'>n</vcpu>
<os><type arch='aarch64' machine='virt-6.2'>hvm</type>
</os>
...
<domain>
```

>![](public_sys-resources/icon-note.gif) **说明：** 
>
>- placement的值必须是static。
>- m为虚拟机当前CPU数目，即虚拟机启动后默认的CPU数目。n为虚拟机支持热插到的最大CPU数目，该值不能超过Hypervisor支持的虚拟机最大CPU规格及GuestOS支持的最大CPU规格，n≥m。
>- 通过virt-install命令创建虚拟机的场景下，默认虚拟机创建完成后，其xml配置文件中不含以上vcpu节点。通过virsh edit <vm name>命令进行动态添加时，该节点不会即时生效，因此添加操作完成后需自行重启虚拟机，才能进行后续的vCPU热插拔操作。

openEuler 24.03 LTS版本中，该特性默认开启，系统安装即使能。非openEuler内核需要自行合入并适配如下功能补丁。

```txt
qemu：
https://gitee.com/openeuler/qemu/pulls/804
https://gitee.com/openeuler/qemu/pulls/850
https://gitee.com/openeuler/qemu/pulls/860
https://gitee.com/openeuler/qemu/pulls/863
guest OS：
https://gitee.com/openeuler/kernel/pulls/4219
https://gitee.com/openeuler/kernel/pulls/5555
https://gitee.com/openeuler/kernel/pulls/7902
https://gitee.com/openeuler/kernel/pulls/9746
```

### 特性验证<a name="ZH-CN_TOPIC_0000002550007409"></a>

在支持的openEuler版本中，部署libvirt 9.10.0及QEMU 8.2.0，并在配置虚拟机xml后，启动虚拟机。以下假设配置虚拟机当前CPU数目为4，所支持的最大CPU数目为8，且虚拟机名为test\_vm。

1. 进入虚拟机。

    ```shell
    virsh console test_vm
    ```

2. 通过**lscpu**命令，记录在线CPU数目，应为4。
3. 通过“ctrl + \]”快捷键退出虚拟机。
4. 执行vCPU热插命令。

    ```shell
    virsh setvcpus test_vm --count 8 --live
    ```

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >- 热插拔命令中的8是热插拔操作后预期的vCPU个数，可按需要自行配置，但不应大于虚拟机定义时预先给定的最大vCPU数目。
    >- --live表示vCPU热插操作即时生效，但重启后会恢复初始设置。
    >- 仅指定--live的情况下，虚拟机一旦重启，热插拔动作将失效。如需将vCPU热插动作持久化，可指定--config参数，使操作在下次虚拟机重启时仍生效。在openEuler 24.03 LTS系统中，config参数与live参数可同时指定。
    >- 虚拟机能通过热插达到的最大vCPU个数受系统限制，且在不同的系统版本限制的个数有所不同。热插命令中指定的vCPU数目如果超过虚拟机定义时预先给定的最大vCPU数目或者系统指定的虚拟机最大vCPU数目（常见于起大规格虚拟机场景下），热插命令会报错，且无法生效。
    >- **lscpu**命令支持查看当前系统内vCPU的在线、离线情况，其中，仅在线状态的vCPU会被虚拟机调度使用。因此，上述验证方法中，所记录的CPU数目，应为当前时刻下，**lscpu**命令回显信息中，"On-line CPU\(s\) list"子项中所含的CPU的个数，也即在线CPU个数。

5. 重新进入虚拟机。

    ```shell
    virsh console test_vm
    ```

6. 通过**lscpu**命令，记录在线CPU数目，应为8，即vCPU热插成功。

## 缩略语<a name="ZH-CN_TOPIC_0000002518687570"></a>

|**缩略语**|**英文全称**|**中文全称**|
|--|--|--|
|RAS|Reliability, Availability and Serviceability|可靠性、可用性和可维护性|
