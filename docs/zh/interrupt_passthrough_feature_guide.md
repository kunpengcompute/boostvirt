# 中断直通 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002043771628"></a>

本文主要介绍如何在使用openEuler操作系统的鲲鹏服务器上使能中断直接注入虚拟机技术以辅助加速虚拟机的性能。

**简介<a name="section10479103292114"></a>**

在传统的虚拟化环境中，中断处理通常需要VMM（Virtual Machine Monitor）的介入。当物理设备产生中断时，中断信号会先被VMM捕获，然后通过软件模拟的方式注入到虚拟机中。然而，这种机制会引入额外的延迟和性能损耗。

中断直接注入技术允许虚拟机直接接收和处理来自物理设备的中断，而无需通过VMM进行复杂的模拟和转发。这种技术通过硬件加速和软件优化，减少了中断处理的延迟和开销，从而显著提升虚拟机的性能。

GIC（General Interrupt Controller，通用中断控制器） v4.1是ARM通用中断控制器的一个高级版本，通过支持虚拟中断的直接注入（包括vLPI和vSGI），显著提高了虚拟化环境中的中断处理效率。

虚拟化硬件辅助加速特性通过在920新型号处理器使能GICv4.1的直接注入虚拟中断和直接注入vSGI的中断直通类型，可以显著降低中断响应时延，提升网络/IO密集型业务吞吐量。

**版本支持<a name="section14446935195215"></a>**

- 支持openEuler 22.03 LTS SP3及以上的操作系统版本。
- 非上述支持操作系统版本的内核需要合入[使能特性](#使能特性)中给出的使能补丁。

**约束与限制<a name="section1683711556528"></a>**

只支持鲲鹏920新型号处理器。

**应用场景<a name="section018355352116"></a>**

使用场景：网络和IO密集型业务，中断较多的时候，可降低中断响应时延，提升业务吞吐量。

鲲鹏920新型号处理器支持的中断直通类型如下：

|中断直通类型|**原理&效果**|**适用场景**|
|--|--|--|
|GICv4.1直通设备vLPI中断透传|基于GICv4.0/4.1，支持VFIO（Virtual Function I/O，虚拟功能I/O）直通设备中断直接注入到VM，无需VM-Exit/Entry。|降低中断响应时延，提升网络/IO密集型业务吞吐量。|
|GICv4.1虚拟设备中断透传|基于GICv4.1+鲲鹏自研优化，支持virtio虚拟设备中断直接注入到VM，无需VM-Exit/Entry。|降低中断响应时延，提升网络/IO密集型业务吞吐量。|
|GICv4.1 vSGI中断透传|基于GICv4.1，支持接收端vCPU核间中断直接注入到VM，无需VM-Exit/Entry。|降低IPI（Inter-Processor Interrupt，处理器间中断）中断响应时延，提升多核场景IPI中断密集业务的性能。|




## 环境要求<a name="ZH-CN_TOPIC_0000002043929896"></a>

本文基于openEuler操作系统提供指导，在正式操作前请确保软硬件均满足要求。

**硬件要求<a name="section26241127"></a>**

硬件要求如[**表 1** 硬件要求](#硬件要求)所示。

**表 1** 硬件要求<a id="硬件要求"></a>

|项目|说明|
|--|--|
|处理器|鲲鹏920新型号处理器|


**操作系统和软件要求<a name="section153345522323"></a>**

操作系统和软件要求如[**表 2** 操作系统和软件要求](#操作系统和软件要求)所示。

**表 2** 操作系统和软件要求<a id="操作系统和软件要求"></a>

|项目|版本|获取方法|
|--|--|--|
|OS|openEuler 22.03 LTS SP3|[获取链接](https://mirrors.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP3/ISO/aarch64/openEuler-22.03-LTS-SP3-everything-aarch64-dvd.iso)|
|libvirt|6.2.0|利用Yum工具直接安装。|
|QEMU|6.2.0|利用Yum工具直接安装。|



## 使能特性<a name="ZH-CN_TOPIC_0000002043771620"></a>

本章节介绍如何在鲲鹏920新型号处理器上使能中断直通特性。

**GICv4.1直通设备vLPI中断透传<a name="section9302349144514"></a>**

1. 通过BIOS使能SMMU。

    进入“BIOS-\>Advanced-\>MISC Config-\>Support SMMU”，设置“Support SMMU”为“Enabled”。

2. BIOS默认配置是GICv4.0，需要手动设置为GICv4.1。

    进入“BIOS-\>Advanced-\>Processor Configuration-\>GIC Version”，设置“GIC Version”为“4.1”。

3. 在Host OS的系统引导配置文件grub.cfg（以openEuler为例，该文件在“/boot/efi/EFI/openEuler/grub.cfg”）中添加如下参数至对应内核（此处以openEuler 22.03 LTS SP3为例）的“linux   /vmlinuz-5.10.0-182.0.0.95.oe2203sp3.aarch64 root=”行尾来启动GICv4.1。添加完参数后重启Host OS生效。

    ```
    kvm-arm.vgic_v4_enable=1
    ```

4. Host执行。

    ```
    dmesg | grep GIC
    ```

    观察到回显有如下信息说明GICv4.1已经使能。

    ```
    kvm [1]: GICv4.1 support enabled
    ```

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >开启GICv4.1后，对直通设备的虚拟机内部进行FIO加压，可以观察到虚拟机内部有很多IO中断，但Host侧对应直通设备的VFIO中断为0。

**GICv4.1 vSGI中断透传<a name="section1016782620462"></a>**

1. 通过BIOS使能SMMU。

    进入“BIOS-\>Advanced-\>MISC Config-\>Support SMMU”，设置“Support SMMU”为“Enabled”。

2. BIOS默认配置是GICv4.0，需要手动设置为GICv4.1。

    进入“BIOS-\>Advanced-\>Processor Configuration-\>GIC Version”，设置“GIC Version”为“4.1”。

3. 在Host OS的系统引导配置文件grub.cfg（以openEuler为例，该文件在/boot/efi/EFI/openEuler/grub.cfg）中添加如下参数至对应内核（此处以openEuler 22.03 LTS SP3为例）的“linux   /vmlinuz-5.10.0-182.0.0.95.oe2203sp3.aarch64 root=”行尾来启动GICv4.1。添加完参数后重启Host OS生效。

    ```
    kvm-arm.vgic_v4_enable=1
    ```

4. Host执行。

    ```
    dmesg | grep GIC
    ```

    观察到回显有如下信息说明GICv4.1已经使能。

    ```
    kvm [1]: GICv4.1 support enabled
    ```

5. 检查GuestOS查看dmesg日志，观察到回显有如下信息说明vSGI透传功能已经使能。

    ```
    Enabling SGIs without active state
    ```

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >在虚拟机内部触发大量IPI中断，vmtop可以观察到虚拟机中断退出基本没有变化。



**GICv4.1虚拟设备中断透传<a name="section4441112154714"></a>**

1. BIOS默认配置是GICv4.0，需要手动设置为GICv4.1。

    进入“BIOS-\>Advanced-\>Processor Configuration-\>GIC Version”，设置“GIC Version”为“4.1”。

2. 在Host OS的系统引导配置文件grub.cfg（以openEuler为例，该文件在/boot/efi/EFI/openEuler/grub.cfg）中添加如下参数至对应内核（此处以openEuler 22.03 LTS SP3为例）的“linux   /vmlinuz-5.10.0-182.0.0.95.oe2203sp3.aarch64 root=”行尾来启动GICv4.1。添加完参数后重启Host OS生效。

    ```
    kvm-arm.vgic_v4_enable=1
    ```

3. Host执行。

    ```
    dmesg | grep GIC
    ```

    观察到回显有如下信息说明GICv4.1已经使能。

    ```
    kvm [1]: GICv4.1 support enabled
    ```

4. 在Host OS的系统引导配置文件grub.cfg（以openEuler为例，该文件在/boot/efi/EFI/openEuler/grub.cfg）中添加如下参数至对应内核（此处以openEuler 22.03 LTS SP3为例）的“linux   /vmlinuz-5.10.0-182.0.0.95.oe2203sp3.aarch64 root=”行尾来给虚拟设备预留PCI号。添加完参数后重启Host OS生效。

    其中start为起始编号，count为PCI号数量。

    ```
    kvm-arm.virt_msi_bypass=1 irqchip.gicv3_rsv_buses_start=30 irqchip.gicv3_rsv_buses_count=10
    ```

    创建虚拟机时，Host OS上dmesg观察到回显有如下信息说明虚拟设备中断卸载使能成功。

    ```
    Create shadow device
    ```

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >在虚拟机内部对虚拟磁盘或虚拟网卡进行加压，vmtop可以观察到vm的中断退出次数基本没有变化。

**非openEuler内核使能补丁<a name="section1763194621512"></a>**

该特性支持openEuler 22.03-LTS-SP3及以上的操作系统版本，非openEuler内核需要合入如下使能补丁。

>![](public_sys-resources/icon-note.gif) **说明：** 
>由于非openEuler内核需要合入的使能补丁数量比较大，建议使用openEuler 22.03-LTS-SP3及以上的操作系统使能该特性。

```
https://lore.kernel.org/all/20191017113341.13778-1-ben.dooks@codethink.co.uk/
https://lists.cs.columbia.edu/pipermail/kvmarm/2019-November/037998.html
https://lore.kernel.org/linux-arm-kernel/20191025135144.8805-1-christoffer.dall@arm.com/T/
https://lore.kernel.org/linux-kernel//7055e836-cdad-1cfa-66f3-fba88dad5f5b@huawei.com/
https://patchwork.kernel.org/project/kvm/patch/20191107160412.30301-2-maz@kernel.org/
https://patchwork.kernel.org/project/kvm/patch/20191107160412.30301-3-maz@kernel.org/
https://lore.kernel.org/all/20191108165805.3071-2-maz@kernel.org/
https://lists.cs.columbia.edu/pipermail/kvmarm/2019-September/037231.html
https://lore.kernel.org/lkml/86ftj7ybg2.wl-maz@kernel.org/
https://lists.cs.columbia.edu/pipermail/kvmarm/2019-October/037746.html
https://lists.cs.columbia.edu/pipermail/kvmarm/2019-September/037236.html
https://lists.cs.columbia.edu/pipermail/kvmarm/2019-September/037237.html
https://lore.kernel.org/lkml/157989306987.396.7210470882766081674.tip-bot2@tip-bot2/
https://lore.kernel.org/all/20191108165805.3071-9-maz@kernel.org/
https://lore.kernel.org/all/20191108165805.3071-10-maz@kernel.org/
https://lore.kernel.org/linux-arm-kernel/86v82vl34a.wl-maz@kernel.org/T/
https://lore.kernel.org/all/20191108165805.3071-12-maz@kernel.org/
https://lore.kernel.org/lkml/157989306987.396.7210470882766081674.tip-bot2@tip-bot2/
https://lore.kernel.org/all/158551357385.28353.9119028347947412581.tip-bot2@tip-bot2/
https://lists.cs.columbia.edu/pipermail/kvmarm/2019-October/037751.html
https://lore.kernel.org/lkml/20191027144234.8395-9-maz@kernel.org/
https://lore.kernel.org/all/3f1aad5c7f79b5ae5b87cef57523ec78@kernel.org/
https://lists.cs.columbia.edu/pipermail/kvmarm/2020-January/038923.html
https://lore.kernel.org/all/20200206075711.1275-2-yuzenghui@huawei.com/
https://lore.kernel.org/all/20200206075711.1275-3-yuzenghui@huawei.com/
https://lore.kernel.org/lkml/20200204090940.1225-4-yuzenghui@huawei.com/
https://lore.kernel.org/lkml/20200206075711.1275-5-yuzenghui@huawei.com/
https://lore.kernel.org/lkml/20200206075711.1275-6-yuzenghui@huawei.com/
https://lore.kernel.org/lkml/20200206075711.1275-7-yuzenghui@huawei.com/
https://kernel.googlesource.com/pub/scm/linux/kernel/git/morse/linux/+/490d332ea42780577f679f5d13598b195bff360c%5E%21/
https://lore.kernel.org/all/158551358355.28353.15317619271465246875.tip-bot2@tip-bot2/
https://lore.kernel.org/all/158551358020.28353.12068573166973176178.tip-bot2@tip-bot2/
https://lore.kernel.org/all/158551357719.28353.15287980899679490964.tip-bot2@tip-bot2/
https://lore.kernel.org/all/158551356984.28353.11059083385737598395.tip-bot2@tip-bot2/
https://lore.kernel.org/all/158711740603.28353.4181366508646401603.tip-bot2@tip-bot2/
https://lore.kernel.org/all/158711740558.28353.444096820434458271.tip-bot2@tip-bot2/
https://lore.kernel.org/all/20200501101204.364798-4-maz@kernel.org/
https://lore.kernel.org/all/159351191045.4006.6389349419327393336.tip-bot2@tip-bot2/
https://lore.kernel.org/all/20200720092328.708-1-yuzenghui@huawei.com/
https://lore.kernel.org/all/20200714184118.416543229@linuxfoundation.org/
https://lore.kernel.org/all/20200630133746.816-1-yuzenghui@huawei.com/
https://lore.kernel.org/all/20230617073242.3199746-1-maz@kernel.org/
https://lore.kernel.org/all/20240221130005.105806928@linuxfoundation.org/
https://lore.kernel.org/all/171265419457.10875.5533397957548411107.tip-bot2@tip-bot2/
https://lore.kernel.org/all/20220124184039.425602485@linuxfoundation.org/
https://lore.kernel.org/all/20201128141857.983-3-lushenming@huawei.com/
https://lore.kernel.org/all/20220412063000.988387489@linuxfoundation.org/
https://lore.kernel.org/all/20200218210736.16432-3-sean.j.christopherson@intel.com/ 
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f8af4519dfb6156045173f38cbd528c043fb25e2
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=8e01d9a396e6db153d94a6004e6473d9ff251a6a
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=5c401308017f256ae9de804b4a1c65be1d390571
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=bad36e4e8cdc9048948490293efefdbd85c40ecc
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=5bd90b0989731520f2cdcfbbe467f1271f3cc803
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ef2e78ddadbb939ce79553b10dee0131d65d8f3e
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=898aa5ce6158c5ccfc256bfc17963bc81981eef8
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2f4f064b31315c7c8986522cf38ef6d11fb77986
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=425c09be0f091a3b5940261d9a5d8467d62987e7
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=0dd57fed6b46659b2db1156cb9100fbcfef6fe5d
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ffedbf0cba153c91a0da5d1280a5e639664c5ab3
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=576a83429757999f220f36f206044af2b9026672
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=c1d4d5cd203cc8ec83d67d4e2af51f1a9f01ba34
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=093bf439fee0d40ade7e309c1288b409cdc3b38f
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=286146960a110cdae455a18cef47d5113d9a95c6
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ed0e4aa9cc74c08a429f4a389483c1076da2ca51
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=046b5054f56691c7f5861197a812f3990f66b30e
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=11635fa26dc7a715f3fc1c351846859e90985ae1
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b2cb11f4f7643255b7703c0fcabc31a8ec478f3a
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f2d834092ee276610ccb6637e5109b61fc79ab89
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b25319d279b63781b972c4966b4082193e69afac
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=5e5168461c22c8738d31d4ee12a5cbc2ab0aa440
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d1bd7e0ba533a2a6f313579ec9b504f6614c35c4
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=64edfaa9a2342a3ce34f8cb982c2c2df84db4de3
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=0684c7046590dd1e8047e187aaf4c7910cc35bce
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=dd3f050a216ef7c8ce21ba48fd3b2ece2155382f
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=29c647f3b5ae1a20221d477442dcdf058cea4a21
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d97c97baa214486cc3d64c996a2214475f6cc83c
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=91bf6395f7b8614a5a9934a0ae9c8b5312d77b29
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e64fab1a1477dbf0c355691914511612ba312932
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b4a4bd0f2629ec2ece7690de1b4721529da29871
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=3858d4dfdfb845e51ee8b4045f61ccba2c3111ee
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f4a81f5a853e0b7c38bfad3afd6d0365d654e777
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=821c10c2ae0bac5a8503cc7e961e7af90ea676eb
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e88bd316e5971fe78884ad1f466b9fc576575e5f
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=8b718d403c5cdc7f0ea492c33ec88169f3e76462
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4e6437f12d6e929e802f5599a2d50dfcf92d0f50
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4bccf1d715fe8f5fe10bd6202c8caa0ae6104cd2
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b46353250ba3b4946adcbbabead23546fcb758b0
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=5186a6cc3ef5a3fa327c258924ef098b0de77006
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=490d332ea42780577f679f5d13598b195bff360c
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=13ea525517088b20399aeb410d9fc567741ac27f
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=04d80dbe858d801efbecf3e5172b31b0a3757308
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d5df9dc96eb7423d3f742b13d5e1e479ff795eaa
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=0b04758b002bde9434053be2fff8064ac3d9d8bb
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=28d160de5194c68ff534443d2a8b6f1d10d57c58
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f3a059219bc718ccc3bf3ff894f089b7e9a93139
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=af9acbfc2c4b72c378d0b9a2ee023ed01055d3e2
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b978c25f6ee7d4c79cbe918eed684e53887ec001
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=9058a4e980648e7d068a7f7726a8ea4c67d0e88a
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=3af9571cd585efafc2facbd8dbd407317ff898cf
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=926846a703cbf5d0635cc06e67d34b228746554b
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=3c40706d05fdea421e991da50e72a29d41131a66
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=5e46a48413a6660955de7e56f9f364f2b890381c
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=166cba71818cd49d7d815fdc6f97c63395e94fc5
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e252cf8a34d92adf41124cb59b19b49d25395548
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b4e8d644ec623cbb66f192a7fefbd0a66e314be8
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=7017ff0ee1de9d45fafee88a4e7890cce92f482e
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=05d32df13c6b3c0850b68928048536e9a736d520
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ae699ad348cdcd416cbf28e8a02fc468780161f7
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=a3f574cd65487cd993f79ab235d70229d9302c1e
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=6d31b6ff985dbd144b2c4d519cf573b8f81865d9
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d50676f5ce8481b98f9bbc1514b5d3f8747dd3c2
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=009384b38034111bf2c0c7bfb2740f5bd45c176c
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=9879b79aefe5ca3cfee0138189c8116f3a71e770
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ef1820be47773012d7526abb8c79befcc1d7b607
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=bacf2c60548befa8a31c2f19ef65bf2177fda33f
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2291ff2f2a569b7b7a64aff3d05e1d51fe0fd66a
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d9c3872cd2f86b7295446e35b4801270669d2960
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=7bdabad12784cf03d2ba36fc7dec66d4f2bb3174
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=dab4fe3bf6dd87a7d6dbab2c929afd1ef62a120b
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=eeaa4b24e5032707ee4286b6a2bcc5fb85eba4a4
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=96806229ca033f85310bc5c203410189f8a1d2ee
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4b2dfe1e7799d0e20b55711dfcc45d2ad35ff46e
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=31dbb6b1d025506b3b8b8b74e9b697df47b9f696
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=46135d6f878ab00261d4a2082d620bfb41019aab
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=12df7429213abbfa9632ab7db94f629ec309a58b
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=80317fe4a65375fae668672a1398a0fb73eb9023
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f66b7b151e00427168409f8c1857970e926b1e27
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ef3691683d7bfd0a2acf48812e4ffe894f10bfa8
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=79a7f77b9b154d572bd9d2f1eecf58c4d018d8e2
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=57e3cebd022fbc035dcf190ac789fd2ffc747f5b
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=af27e41612ec7e5b4783f589b753a7c31a37aac8
https://gitee.com/openeuler/kernel/commit/3662006884d195d0ccbaa30818eea04dc9928716
https://gitee.com/openeuler/kernel/commit/7b9b5990e5f2d5d14cbb782c9f5ae88a30932fe4
https://gitee.com/openeuler/kernel/commit/ccd1c07f6d9b662657d044e4525a1964ec5afb4e
https://gitee.com/openeuler/kernel/commit/8544c8e14fec52453d0b38a4b6bdf12345e71755
https://gitee.com/openeuler/kernel/pulls/3220/commits
https://gitee.com/openeuler/kernel/commit/539979b6ec62f7bba40b0452b0574a0f4ec4fe4e
https://gitee.com/openeuler/kernel/commit/ba1ed9e17b581c9a204ec1d72d40472dd8557edd
https://gitee.com/openeuler/kernel/commit/41ee52ecbcdc98eb2a03241923b1a7d46f476bb3
https://gitee.com/openeuler/kernel/commit/43c7519a5d563325b19210009d04643fedd96204
```

**合入补丁示例，以git am方式合入补丁为例：**

1. 以git am方式合入补丁。

    ```
    git am -s  /path/to/patch/Reinstall_old_memslots_if_arch_preparation_fails.patch
    ```

2. 若只提示applying成功信息，则补丁可用，回显信息如下。

    ```
    Applying：KVM：Reinstall old memslots if arch preparation fails
    ```

3. 若有如下报错，则需根据错误提示在patch中找到无法合入部分，根据提示到相应文档中手动修改。

    ```
    Applying：KVM：Reinstall old memslots if arch preparation fails
    error：patch failed：virt/kvm/kvm_main.c:1117
    error：virt/kvm/kvm_main.c：patch does not apply
    Patch failed at 0001 KVM：Reinstall old memslots if arch preparation fails
    ```

4. 补丁合入并解决冲突后，重新编译安装内核。


