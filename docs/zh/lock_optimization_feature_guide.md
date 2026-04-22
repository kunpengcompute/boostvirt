# 虚拟机锁优化 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002106021525"></a>

### 简介<a name="ZH-CN_TOPIC_0000002070181886"></a>

本文介绍虚拟机锁优化特性的工作原理、适用场景、环境要求及特性使能详细步骤。

虚拟机锁是一种保护机制，用于管理对虚拟机的访问和操作，确保虚拟机的资源在特定时间内只能被一个用户或进程独占。然而，在超分场景下，容易出现多个虚拟机抢占同一个虚拟机锁的情况，导致虚拟机性能下降。

虚拟机锁优化特性对抢占过程中的锁机制进行了优化，以提升虚拟机在超分场景下的性能。虚拟机侧操作系统在申请锁时，可通过共享内存查询对应的vCPU是否已经被其他虚拟机抢占。如果vCPU已经被抢占，则退出锁等待；如果vCPU未抢占，则进入锁等待。

虚拟机锁优化特性通过共享内存的方式，将vCPU是否被抢占的信息通过Hypervisor透传给虚拟机，虚拟机vCPU自旋等待时如果发现持锁的vCPU已经被抢占，则跳出自旋等待，减少因操作冲突导致的系统错误或崩溃，从而提高虚拟机系统稳定性和可靠性。

**图 1** 虚拟机锁优化原理图<a name="fig2050617656"></a><a id="虚拟机锁优化原理图"></a>
![](figures/虚拟机锁优化原理图.png "虚拟机锁优化原理图")

1. 虚拟机内核启动时，通过Hypercall将每个vCPU记录preempted状态的内存地址传递给Hypervisor。
2. 当Hypervisor调度运行vCPU时，调用kvm\_arch\_vcpu\_load加载对应vCPU上下文，同时将其preempted状态设置为0。
3. 当Hypervisor调度vCPU停止运行、或切换到另一个vCPU时，将调用kvm\_arch\_vcpu\_put保存对应vCPU上下文，同时将其preempted状态设置为1。
4. 虚拟机内部vCPU在申请锁时（mutex、rwsem、osq\_lock等场景），如果发现持有锁的vCPU已经被调度停止（其preempted状态为1），则申请锁的vCPU跳出自旋等待。

其中，preempted状态值用于记录vCPU在Host上是否被抢占，0表示Hypervisor正在调度执行该vCPU，1表示该vCPU已被Hypervisor调度停止。


### 可获得性<a name="ZH-CN_TOPIC_0000002105901569"></a>

版本支持：

- 支持openEuler 20.03 LTS SP1及以上的操作系统版本。
- 非openEuler内核需要合入如[**表 3** 非openEuler内核补丁文件要求](#非openEuler内核补丁文件要求)所示使能补丁。补丁合入默认特性使能。


### 约束与限制<a name="ZH-CN_TOPIC_0000002070341630"></a>

虚拟机锁优化特性暂不支持热迁移。

虚拟机锁优化特性仅与操作系统内核有关，特性使能与否对libvirt和QEMU完全透明，无需在libvirt及QEMU侧进行任何附加操作。


### 应用场景<a name="ZH-CN_TOPIC_0000002105901593"></a>

1. 虚拟机锁优化适用于vCPU范围绑核的超分场景，即一个物理核可能会有多个vCPU线程争抢资源的情况。
2. 本特性以前后端协同方式，获取vCPU线程的抢占状态，优化内部调度/锁的性能。在涉及到CPU调度的场景下，如mutex\_spin\_on\_owner、mutex\_can\_spin\_on\_owner、rtmutex\_spin\_on\_owner and osq\_lock、available\_idle\_cpu会用到本特性。



## 特性使用<a name="ZH-CN_TOPIC_0000002070341634"></a>

### 环境要求<a name="ZH-CN_TOPIC_0000002154855905"></a>

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
|OS|openEuler 20.03 LTS SP1<br>物理机、虚拟机均要求操作系统不低于以上版本|[获取链接](https://mirrors.huaweicloud.com/openeuler/openEuler-20.03-LTS-SP1/ISO/aarch64/openEuler-20.03-LTS-SP1-aarch64-dvd.iso)|
|libvirt|6.2.0及以上均可|利用Yum工具直接安装。|
|QEMU|4.1.0及以上均可|利用Yum工具直接安装。|


**表 3** 非openEuler内核补丁文件要求<a id="非openEuler内核补丁文件要求"></a>

|序号|补丁名称|获取方法|
|--|--|--|
|1|KVM: arm64: Document PV-sched interface|[获取链接](https://gitee.com/openeuler/kernel/commit/b74edaf629bdd6bb66d8852b783812560b29079c)|
|2|KVM: arm64: Implement PV_SCHED_FEATURES call|[获取链接](https://gitee.com/openeuler/kernel/commit/a0b95bdf6a0b2d5a96a28b1f728a6abad51dbaec)|
|3|KVM: arm64: Support pvsched preempted via shared structure|[获取链接](https://gitee.com/openeuler/kernel/commit/76732c97a3ecc07a4237f0348f661ccff6b9d3eb)|
|4|KVM: arm64: Add interface to support vCPU preempted check|[获取链接](https://gitee.com/openeuler/kernel/commit/63042c58affca22fb14736455d38d2a6e97ca9dc)|
|5|KVM: arm64: Support the vCPU preemption check|[获取链接](https://gitee.com/openeuler/kernel/commit/cf6d95e33dfa0344abea230ed48a1fad69591f36)|
|6|KVM: arm64: Add SMCCC PV-sched to kick cpu|[获取链接](https://gitee.com/openeuler/kernel/commit/7a645f6e24eeda4736975b6e85dc797a57fe8900)|
|7|KVM: arm64: Implement PV_SCHED_KICK_CPU call|[获取链接](https://gitee.com/openeuler/kernel/commit/efed88dd593493653466917a7e95868ec38bef41)|
|8|KVM: arm64: Add interface to support PV qspinlock|[获取链接](https://gitee.com/openeuler/kernel/commit/12e1ed766c347d47f85736fece113c7578ace94f)|


>![](public_sys-resources/icon-notice.gif) **须知：** 
>本特性功能需要物理机、虚拟机侧都有相应功能支持。因此，要在非openEuler内核上使用该特性，则物理机、虚拟机上都要合入以上补丁。


### 特性使能<a name="ZH-CN_TOPIC_0000002105901585"></a>

该特性支持openEuler 20.03 LTS SP1及以上的操作系统版本，在这些系统上默认使能，无需操作。

在非openEuler内核上，使能该特性需要合入相应补丁，见[**表 3** 非openEuler内核补丁文件要求](#非openEuler内核补丁文件要求)，补丁合入后，系统启动默认使能。


### 特性验证<a name="ZH-CN_TOPIC_0000002070181858"></a>

在安装libvirt和QEMU的前提下，创建并启动虚拟机。虚拟机系统启动后，在虚拟机内部执行如下命令。

```
dmesg | grep PV
```

如果特性运转正常，回显中将含有如下内容。

```
arm-pv: using PV sched preempted
```

本特性对用户完全透明，无需任何操作，即可提升虚拟机的vCPU调度性能。



## 缩略语<a name="ZH-CN_TOPIC_0000002106021549"></a>

|**缩略语**|**英文全称**|**中文全称**|
|--|--|--|
|KVM|Kernel-based Virtual Machine|内核虚拟机|



