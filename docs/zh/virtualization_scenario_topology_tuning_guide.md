# 虚拟化场景拓扑调优指南

## 调优概述<a name="ZH-CN_TOPIC_0000002518687258"></a>

本文针对基于KVM的虚拟化场景，指导用户对鲲鹏服务器的虚拟化参数配置进行适当调整，以实现虚拟化场景下的最佳性能表现。

在虚拟化场景中，多个虚拟机共享同一物理机的CPU核心资源，资源竞争可能导致性能瓶颈，影响虚拟机的性能表现。通过将虚拟机的vCPU与物理CPU核心绑定，可以合理分配资源，避免资源过度集中或不足，从而提升整体性能。在服务器CPU架构中，Socket中包含多个CPU Die，而CPU Die中包含多个Cluster。因此，虚拟机vCPU绑定的物理CPU核心所在的微架构不同，会影响vCPU所使用的资源范围。

>![](public_sys-resources/icon-note.gif) **说明：** 
>-   Socket是物理层级的CPU封装单位，对应一个独立的处理器封装。
>-   Die是指中央处理器（CPU）芯片上的芯片核心区域。它是一个单独的硅片，其中包含了CPU的核心部分，包括运算单元、缓存、控制逻辑等。
>-   Cluster是芯片内部多个CPU核心的逻辑分组，共享缓存等资源。


## 环境要求<a name="ZH-CN_TOPIC_0000002550007099"></a>

描述被调优的服务器的硬件要求和软件要求。

**硬件要求<a name="section1815816277716"></a>**

硬件要求如[**表 1** 硬件要求](#硬件要求)所示。

**表 1** 硬件要求<a id="硬件要求"></a>

|项目|物理机配置|
|--|--|
|服务器|鲲鹏服务器|
|处理器|2*鲲鹏920新型号处理器|
|内存|内存按1DPC方式配置将获得最佳性能，即将DIMM0插满。|
|系统盘|无特殊要求|


**软件要求<a name="section102619448716"></a>**

软件要求如[**表 2** 软件要求](#软件要求)所示。

**表 2** 软件要求<a id="软件要求"></a>

|项目|软件版本|获取方式|
|--|--|--|
|操作系统|openEuler 24.03 LTS SP1|[获取链接](https://mirrors.huaweicloud.com/openeuler/openEuler-24.03-LTS-SP1/ISO/aarch64/openEuler-24.03-LTS-SP1-everything-aarch64-dvd.iso)|
|libvirt|9.10.0|yum源安装|
|QEMU|8.2.0|yum源安装|



## 4个NUMA节点场景配置Cluster调优<a name="ZH-CN_TOPIC_0000002518527338" id="4个NUMA节点场景配置Cluster调优"></a>

通过对虚拟机vCPU所绑定的物理CPU核心和vCPU拓扑进行Cluster调优，以确保虚拟机的最佳性能表现。

为了提升虚拟机的性能，通常会对虚拟机vCPU指定物理CPU核心和指明虚拟机的vCPU拓扑。本文建议在虚拟机的vCPU与物理CPU核心进行1:1绑定，并确保绑定的物理CPU核心所在的Cluster数最小化，以达到最佳性能。

>![](public_sys-resources/icon-note.gif) **说明：** 
>该示例基于32C64G规格的虚拟机进行Cluster调优配置，请根据实际需求和虚拟机规格对参数进行相应的调整。

1. 找到目标虚拟机名称。

    ```
    virsh list --all
    ```

    ![](figures/zh-cn_image_0000002518687260.png)

2. 修改虚拟机xml。

    ```
    virsh edit <虚拟机名称>
    ```

    以下是一个Cluster调优策略的配置示例。在本示例中，cputune模块中对vCPU与物理CPU核心进行1:1绑定。在numatune模块中，只设置单虚拟机NUMA节点，nodeset指向物理机核心所在NUMA节点。在虚拟机的cpu模块中，设置Socket数量为1，Die数量为1，Clusters数量为4，每个Cluster中的vCPU数量为4，超线程数量为2。

    ```
    <domain type = 'KVM'>
    ...
      <vcpu placement='static'>32</vcpu>
      <cputune>
        <vcpupin vcpu='0' cpuset='8'/>
        <vcpupin vcpu='1' cpuset='9'/>
        <vcpupin vcpu='2' cpuset='10'/>
        <vcpupin vcpu='3' cpuset='11'/>
        <vcpupin vcpu='4' cpuset='12'/>
        <vcpupin vcpu='5' cpuset='13'/>
        <vcpupin vcpu='6' cpuset='14'/>
        <vcpupin vcpu='7' cpuset='15'/>
        <vcpupin vcpu='8' cpuset='16'/>
        <vcpupin vcpu='9' cpuset='17'/>
        <vcpupin vcpu='10' cpuset='18'/>
        <vcpupin vcpu='11' cpuset='19'/>
        <vcpupin vcpu='12' cpuset='20'/>
        <vcpupin vcpu='13' cpuset='21'/>
        <vcpupin vcpu='14' cpuset='22'/>
        <vcpupin vcpu='15' cpuset='23'/>
        <vcpupin vcpu='16' cpuset='24'/>
        <vcpupin vcpu='17' cpuset='25'/>
        <vcpupin vcpu='18' cpuset='26'/>
        <vcpupin vcpu='19' cpuset='27'/>
        <vcpupin vcpu='20' cpuset='28'/>
        <vcpupin vcpu='21' cpuset='29'/>
        <vcpupin vcpu='22' cpuset='30'/>
        <vcpupin vcpu='23' cpuset='31'/>
        <vcpupin vcpu='24' cpuset='32'/>
        <vcpupin vcpu='25' cpuset='33'/>
        <vcpupin vcpu='26' cpuset='34'/>
        <vcpupin vcpu='27' cpuset='35'/>
        <vcpupin vcpu='28' cpuset='36'/>
        <vcpupin vcpu='29' cpuset='37'/>
        <vcpupin vcpu='30' cpuset='38'/>
        <vcpupin vcpu='31' cpuset='39'/>
        <emulatorpin cpuset='8-39'/>
      </cputune>
    ...
      <numatune>
        <memnode cellid='0' mode='strict' nodeset='0'/>
      </numatune>
    ...
      <cpu mode='host-passthrough' check='none'>
        <topology sockets='1' dies='1' clusters='4' cores='4' threads='2'/>
     ...
      </cpu>
    ...
    <domain>
    ```


## 2个NUMA节点场景配置CPU调优<a name="ZH-CN_TOPIC_0000002550127095"></a>

### 修改BIOS配置<a name="ZH-CN_TOPIC_0000002550127097"></a>

物理机设置成2个NUMA节点，需要对BIOS配置进行修改。

在BIOS中开启One Numa Per Socket和Die交织选项，如[**表 1** 2个NUMA节点BIOS配置](#2个NUMA节点BIOS配置)所示：

**表 1** 2个NUMA节点BIOS配置<a id="2个NUMA节点BIOS配置"></a>

|选项|配置值|参数路径|
|--|--|--|
|One Numa Per Socket|Enabled|Advanced>Memory Configuration>One Numa Per Socket|
|Die Interleaving|Enabled|Advanced>Memory Configuration>Die Interleaving|


按照上表推荐的BIOS配置项，配置操作步骤如下：

1. 重启服务器，进入BIOS设置界面。

    具体操作请参见《[TaiShan 服务器 BIOS 参数参考（鲲鹏920处理器）](https://support.huawei.com/enterprise/zh/doc/EDOC1100088653/a77cbc34)》中“进入BIOS界面”的相关内容。

2. 开启One Numa Per Socket。

    在BIOS中，依次选择“Advanced\>Memory Configuration\>One Numa Per Socket”，设置为Enabled。

3. 开启Die Interleaving。

    在BIOS中，依次选择“Advanced\>Memory Configuration\>Die Interleaving”，设置为Enabled。

4. 按**F10**保存BIOS设置，并重启服务器。


### CPU绑核调优<a name="ZH-CN_TOPIC_0000002518527340"></a>

在2个NUMA节点场景中，调整虚拟机vCPU绑定的物理机所在Die与vCPU拓扑，确保虚拟机的高性能。

物理机设置成2个NUMA节点后，每个NUMA节点中包含两个CPU Die。两个CPU Die之间的缓存与内存读写存在性能损失，因此建议在虚拟机vCPU绑定的物理CPU核心不能跨越Die。以鲲鹏920新型号处理器7270Z为例，该型号处理器一个CPU Die包含64个CPU核心。如下图所示，存在两个NUMA节点，其中CPU 0-127为NUMA 0节点。而NUMA 0节点包含两个CPU Die，0-63 CPU为第一个CPU Die的核心，64-127 CPU为第二个CPU Die的核心。

![](figures/zh-cn_image_0000002518527342.png)

当虚拟机的规格小于64U时，虚拟机最佳性能的绑核配置为所有vCPU绑定在第一个Die或者第二个Die的物理CPU核心上，vCPU不能同时绑定在两个Die的物理CPU核心上。

以下是一个CPU绑核调优的示例。在本示例中，cputune模块中对vCPU与物理CPU核心进行1:1绑定，并且所有物理CPU核心均处于同一个CPU Die。在numatune模块中，只设置单虚拟机NUMA节点，nodeset指向物理机核心所在NUMA节点。在虚拟机的cpu模块中，设置Socket数量为1，Die数量为1，Clusters数量为4，每个Cluster中的vCPU数量为4，超线程数量为2。修改虚拟机xml操作参考[4个NUMA节点场景配置Cluster调优](#4个NUMA节点场景配置Cluster调优)。

>![](public_sys-resources/icon-note.gif) **说明：** 
>该示例是基于32C64G规格的虚拟机进行CPU绑核调优，请根据实际需求和虚拟机规格对参数进行相应的调整。

```
<domain type = 'KVM'>
...
  <vcpu placement='static'>32</vcpu>
  <cputune>
    <vcpupin vcpu='0' cpuset='64'/>
    <vcpupin vcpu='1' cpuset='65'/>
    <vcpupin vcpu='2' cpuset='66'/>
    <vcpupin vcpu='3' cpuset='67'/>
    <vcpupin vcpu='4' cpuset='68'/>
    <vcpupin vcpu='5' cpuset='69'/>
    <vcpupin vcpu='6' cpuset='70'/>
    <vcpupin vcpu='7' cpuset='71'/>
    <vcpupin vcpu='8' cpuset='72'/>
    <vcpupin vcpu='9' cpuset='73'/>
    <vcpupin vcpu='10' cpuset='74'/>
    <vcpupin vcpu='11' cpuset='75'/>
    <vcpupin vcpu='12' cpuset='76'/>
    <vcpupin vcpu='13' cpuset='77'/>
    <vcpupin vcpu='14' cpuset='78'/>
    <vcpupin vcpu='15' cpuset='79'/>
    <vcpupin vcpu='16' cpuset='80'/>
    <vcpupin vcpu='17' cpuset='81'/>
    <vcpupin vcpu='18' cpuset='82'/>
    <vcpupin vcpu='19' cpuset='83'/>
    <vcpupin vcpu='20' cpuset='84'/>
    <vcpupin vcpu='21' cpuset='85'/>
    <vcpupin vcpu='22' cpuset='86'/>
    <vcpupin vcpu='23' cpuset='87'/>
    <vcpupin vcpu='24' cpuset='88'/>
    <vcpupin vcpu='25' cpuset='89'/>
    <vcpupin vcpu='26' cpuset='90'/>
    <vcpupin vcpu='27' cpuset='91'/>
    <vcpupin vcpu='28' cpuset='92'/>
    <vcpupin vcpu='29' cpuset='93'/>
    <vcpupin vcpu='30' cpuset='94'/>
    <vcpupin vcpu='31' cpuset='95'/>
    <emulatorpin cpuset='64-95'/>
  </cputune>
...
  <numatune>
    <memnode cellid='0' mode='strict' nodeset='0'/>
  </numatune>
...
  <cpu mode='host-passthrough' check='none'>
    <topology sockets='1' dies='1' clusters='4' cores='4' threads='2'/>
...
  </cpu>
...
<domain>
```



## 缩略语<a name="ZH-CN_TOPIC_0000002518687256"></a>

|**缩略语**|**英文全称**|**中文全称**|
|--|--|--|
|NUMA|Non-Uniform Memory Access|非一致性内存访问|
|KVM|Kernel-based Virtual Machine|基于内核的虚拟机|
|DIMM|Dual Inline Memory Module|双列直插内存模块|
|DIMM0|Dual Inline Memory Module slot 0|计算机主板上的第一个内存插槽|



