# vCPU Hotplug Feature Guide<a name="EN-US_TOPIC_0000002521248824"></a>

## Feature Description<a name="EN-US_TOPIC_0000002518527652"></a>

### Overview<a name="EN-US_TOPIC_0000002550127409"></a>

This document describes how to deploy and enable the vCPU hotplug feature on a Kunpeng server running the openEuler OS.

This feature can increase or decrease the number of vCPUs of a running VM without interrupting services.

Nowadays, OSs must run reliably for a long time and have a powerful fault tolerance mechanism, that is, Reliability, Availability, and Serviceability (RAS). Availability requires that minor problems in the system do not affect the normal running of the entire system. In some cases, hot plugging can be performed to replace faulty components, ensuring that the system downtime is within a certain range. In virtualization scenarios, dynamic scheduling of resources is also required in addition to RAS. When creating a VM, the customer cannot accurately predict the pressure of services on device resources. If the pressure on a device in a VM stays too high or too low for a long time, the service running speed is restricted or resources are wasted. vCPU hotplug can effectively solve this problem.

Resource scaling is one of the major advantages of cloud computing, and vCPU hotplug is one of the key technologies to achieve scalable CPU computing power. vCPU hotplug provides the following benefits:

- Accelerated VM startup, particularly for lightweight secure containers such as Kata containers. In this scenario, a container is initially configured with a single vCPU, with additional vCPUs being hot added after the container is started.
- On-demand resource scalability to reduce service costs. Developers can increase or decrease the number of vCPUs online based on service load requirements.


### Constraints<a name="EN-US_TOPIC_0000002550127413"></a>

- For processors using the AArch64 architecture, the specified VM chipset type (machine) needs to be virt-4.2 or a later version when a VM is created. For processors using the x86_64 architecture, the specified VM chipset type (machine) needs to be pc-i440fx-1.5 or a later version when a VM is created.
- When configuring Guest NUMA, you need to configure the vCPUs that belong to the same socket in the same vNode. Otherwise, the VM may be soft locked up after the vCPUs are hot added, and the VM may fail to run properly.
- VMs do not support vCPU hot add during migration, hibernation, wakeup, or snapshot.
- Whether the hot added vCPU can automatically go online depends on the VM OS logic rather than the virtualization layer.
- When a VM is being started, stopped, or restarted, the hot added vCPU may become invalid. However, the hot added CPU takes effect after the VM is restarted.
- When the vCPUs are hot added, if the number of added vCPUs is not an integer multiple of the number of cores in the VM vCPU topology configuration item, the vCPU topology displayed in the VM may be disordered. You are advised to add vCPUs whose number is an integer multiple of the number of cores each time.
- Feature enhancement: vCPU hot add has already been supported in earlier versions of openEuler (for example, openEuler 22.03 SP4). openEuler 24.03 LTS adds support for vCPU hot removal to adapt to more scenarios. However, the hotplug protocol has been modified in openEuler 24.03 LTS and is no longer compatible with earlier versions. This means that to utilize the vCPU hotplug feature, the guest kernel version must match the QEMU version on the host.
- Hotplug specifications
    - The VM XML configuration file must contain the `vcpu` node as required by QEMU. This node is used to specify the number of vCPUs when a VM is started for the first time and the maximum number of vCPUs that can be reached through vCPU hot add.
    - vCPU hot add is restricted by the maximum number of CPUs supported by the hypervisor and guest OS. The specific restriction depends on the OS type.


### Application Scenarios<a name="EN-US_TOPIC_0000002550007407"></a>

- vCPU hotplug can significantly accelerate VM startup, particularly in the lightweight secure container scenario such as Kata secure containers. In this scenario, a container is initially configured with a single vCPU, with additional vCPUs being hot added after the container is started.

- vCPU hotplug enables on-demand scalability for cloud vendors, allowing them to dynamically adjust the number of vCPUs allocated to user VMs as requested, meeting users' service load requirements without disruption to running services.



## Feature Usage<a name="EN-US_TOPIC_0000002518687568"></a>

### Environment Requirements<a name="EN-US_TOPIC_0000002518527648"></a>

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
|OS|openEuler 24.03 LTS|[Link](https://mirrors.pku.edu.cn/openeuler/openEuler-24.03-LTS/ISO/aarch64/openEuler-24.03-LTS-everything-aarch64-dvd.iso)|
|libvirt|9.10.0|Install it using the Yum tool.|
|QEMU|8.2.0|Install it using the Yum tool.|


### Feature Enablement<a name="EN-US_TOPIC_0000002550127411"></a>

To use the vCPU hot add function, configure the number of CPUs, the maximum number of CPUs supported by the VM, and the VM chipset type when creating the VM. (For the AArch64 architecture, the virt-4.2 or a later version is required. For the x86_64 architecture, the pc-i440fx-1.5 or a later version is required.) The AArch64 VM is used as an example. The configuration template is as follows:

```
<domain type='kvm'>
...
<vcpu placement='static' current='m'>n</vcpu>
<os><type arch='aarch64' machine='virt-6.2'>hvm</type>
</os>
...
<domain>
```

>![](public_sys-resources/icon-note.gif) **NOTE:**
>-   The value of `placement` must be `static`.
>-   `m` indicates the current number of CPUs on the VM, that is, the default number of CPUs after the VM is started. `n` indicates the maximum number of CPUs that can be hot added to a VM. The value cannot exceed the maximum CPU specifications supported by the hypervisor or guest OS. `n` is greater than or equal to `m`.
>-   When a VM is created using the `virt-install` command, the XML configuration file does not contain the preceding vCPU node by default after the VM is created. When a node is dynamically added using the `virsh edit <VM_name>` command, the node does not take effect immediately. It is required to restart the VM after the node is added for subsequent vCPU hotplug operations.

In openEuler 24.03 LTS, this feature is enabled by default. For the non-openEuler kernel, integrate and adapt to the following function patches:

```
QEMU:
https://gitee.com/openeuler/qemu/pulls/804
https://gitee.com/openeuler/qemu/pulls/850
https://gitee.com/openeuler/qemu/pulls/860
https://gitee.com/openeuler/qemu/pulls/863
Guest OS:
https://gitee.com/openeuler/kernel/pulls/4219
https://gitee.com/openeuler/kernel/pulls/5555
https://gitee.com/openeuler/kernel/pulls/7902
https://gitee.com/openeuler/kernel/pulls/9746
```


### Feature Verification<a name="EN-US_TOPIC_0000002550007409"></a>

In the supported openEuler versions, deploy libvirt 9.10.0 and QEMU 8.2.0, configure the VM XML file, and start the VM. Assume that the number of CPUs configured for the VM is 4, the maximum number of CPUs supported by the VM is 8, and the VM name is `test_vm`.

1. Log in to the VM.

    ```
    virsh console test_vm
    ```

2. Run `lscpu` and record the number of CPUs. The number should be 4.
3. Press `Ctrl`+`]` to exit the VM.
4. Run the vCPU hot add command.

    ```
    virsh setvcpus test_vm --count 8 --live
    ```

    >![](public_sys-resources/icon-note.gif) **NOTE:**
    >-   In the preceding command, `8` is the expected number of vCPUs after the hotplug operation. You can configure the number as required, but the number cannot be greater than the maximum number of vCPUs specified during VM definition.
    >-   `--live` indicates that the vCPU hot add takes effect immediately, but the initial settings are restored after a restart.
    >-   If only `--live` is specified, the hotplug operation becomes invalid when the VM is restarted. To retain the vCPU hot add operation, specify the `--config` parameter so that the operation still takes effect after the VM is restarted. In openEuler 24.03 LTS, the `config` and `live` parameters can be specified at the same time.
    >-   The maximum number of vCPUs that can be hot added to a VM is restricted by the OS. This number varies according to the OS version. If the number of vCPUs specified in the hot add command exceeds the maximum number of vCPUs specified when the VM is defined or the maximum number of vCPUs specified by the OS (usually in high-specification-VM scenarios), an error is reported and the hot add command does not take effect.
    >-   The `lscpu` command can be used to check whether vCPUs are online or offline in the current system. Only online vCPUs can be scheduled and used by VMs. Therefore, in the preceding verification method, the number of CPUs recorded is the number of CPUs specified by `On-line CPU(s) list` in the command output of the `lscpu` command, that is, the number of online CPUs.

5. Log in to the VM again.

    ```
    virsh console test_vm
    ```

6. Run `lscpu` and record the number of online CPUs. The number should be `8`, indicating that the vCPU hot add is successful.



## Acronyms and Abbreviations<a name="EN-US_TOPIC_0000002518687570"></a>

|**Acronym/Abbreviation**|**Full Spelling**|
|--|--|
|RAS|Reliability, Availability, and Serviceability|Reliability, availability, and serviceability (RAS)|
