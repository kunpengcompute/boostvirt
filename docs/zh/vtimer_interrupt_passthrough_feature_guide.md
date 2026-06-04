# vtimer中断透传 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002043771630"></a>

### 简介<a name="ZH-CN_TOPIC_0000002070181890"></a>

本文主要介绍如何在使用openEuler操作系统的鲲鹏服务器上使能vtimer中断透传特性以降低虚拟机timer中断注入时延。

在传统虚拟化环境中，虚拟机timer中断的处理需要通过VMM（Virtual Machine Monitor）进行中断注入，该过程涉及虚拟机的陷入陷出，引入了额外的时延开销。对于对timer定时时延有极致要求的业务场景，这种开销会显著影响业务性能。

vtimer中断透传特性允许虚拟机的timer中断直接注入，免去中断注入时的陷入陷出，从而减少中断注入时延，适用于鲲鹏920新型号及鲲鹏950处理器。

>由于实现原因，vtimer直通时不提供active、pending状态。若虚拟机内部依赖timer的active、pending状态做特殊处理，可通过在libvirt配置开启vtimer status状态模拟，该特性需要Guest OS支持。

### 可获得性<a name="ZH-CN_TOPIC_0000002105901570"></a>

版本支持：

支持openEuler 24.03 LTS SP4及以上的操作系统版本。

### 约束与限制<a name="ZH-CN_TOPIC_0000002070341631"></a>

- 仅支持鲲鹏920新型号及鲲鹏950处理器。
- vtimer直通时不提供active、pending状态，若虚拟机内部依赖timer的active、pending状态做特殊处理，需开启vtimer status状态模拟。
- vtimer status状态模拟特性需要Guest OS支持。

### 应用场景<a name="ZH-CN_TOPIC_0000002105901594"></a>

适用于对timer定时时延有极致要求的业务场景，如高精度定时任务、实时业务等。通过避免timer中断注入过程中的陷入/陷出操作，从而降低中断注入时延，提升虚拟机timer相关业务的性能。



## 特性使用<a name="ZH-CN_TOPIC_0000002070341635"></a>

### 环境要求<a name="ZH-CN_TOPIC_0000002154855906"></a>

本文基于openEuler操作系统提供指导，在正式操作前请确保软硬件均满足要求。

**硬件要求<a name="section26241128"></a>**

硬件要求如[**表 1** 硬件要求](#硬件要求)所示。

**表 1** 硬件要求<a id="硬件要求"></a>

|项目|说明|
|--|--|
|处理器|鲲鹏920新型号或鲲鹏950处理器|
|BIOS配置|开启GICv4.1或GICv4.2支持|


**操作系统和软件要求<a name="section153345522324"></a>**

操作系统和软件要求如[**表 2** 操作系统和软件要求](#操作系统和软件要求)所示。

**表 2** 操作系统和软件要求<a id="操作系统和软件要求"></a>

|项目|版本|获取方法|
|--|--|--|
|Host OS|openEuler 24.03 LTS SP4及以上版本|利用Yum工具直接安装。|
|Guest OS|openEuler 24.03 LTS SP4及以上版本（开启vtimer status状态模拟时需要）|利用Yum工具直接安装。|
|libvirt|9.10.0及以上均可|利用Yum工具直接安装。|
|QEMU|8.2.0及以上均可|利用Yum工具直接安装。|


### 特性使能<a name="ZH-CN_TOPIC_0000002105901586"></a>

#### 使能vtimer中断透传<a name="section_vtimer_enable"></a>

1. 在Host OS的系统引导配置文件grub.cfg（以openEuler为例，该文件在“/boot/efi/EFI/openEuler/grub.cfg”）中添加如下参数至对应内核的“linux”行尾来启动vtimer中断透传。添加完参数后重启Host OS生效。

    ```
    kvm-arm.vtimer_irqbypass=1
    ```

2. 重启Host OS使配置生效。

    ```
    reboot
    ```

#### 使能vtimer status状态模拟<a name="section_vtimer_status_enable"></a>

>![](public_sys-resources/icon-notice.gif) **须知：**
>vtimer status状态模拟特性需要Guest OS支持。请在确认Guest OS支持该特性后再进行以下配置。

若虚拟机内部依赖timer的active、pending状态做特殊处理，需要在虚拟机libvirt XML配置文件中添加如下配置：

```xml
<features>
  <kvm-vtimer-status enabled='yes'/>
</features>
```

配置完成后，重新定义虚拟机使配置生效。

```
virsh define vm.xml
```


### 特性验证<a name="ZH-CN_TOPIC_0000002070181859"></a>

#### 验证vtimer中断透传

在Host OS上执行如下命令，确认vtimer中断透传特性是否使能：

```
cat /sys/module/kvm_arm/parameters/vtimer_irqbypass
```

若回显为1，说明vtimer中断透传特性已使能。

#### 验证vtimer status状态模拟

在虚拟机内部执行如下命令，确认vtimer status状态模拟特性是否生效：

```
dmesg | grep vtimer
```

若回显中包含`using pvtimer status`状态模拟相关的日志信息，说明特性已生效。



## 缩略语<a name="ZH-CN_TOPIC_0000002106021550"></a>

|**缩略语**|**英文全称**|**中文全称**|
|--|--|--|
|VMM|Virtual Machine Monitor|虚拟机监视器|
|vtimer|Virtual Timer|虚拟定时器|
