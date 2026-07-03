# vtimer Interrupt Passthrough Feature Guide

## Feature Description<a name="EN-US_TOPIC_0000002043771630"></a>

### Overview<a name="EN-US_TOPIC_0000002070181890"></a>

This document describes how to enable the vtimer interrupt passthrough feature on a Kunpeng server running openEuler to reduce the latency of VM timer interrupt injection.

In traditional virtualization environments, VM timer interrupts are injected through the virtual machine monitor (VMM). This process involves VM entry and exit, which introduces additional latency overhead. In service scenarios that have stringent requirements on timer latency, this overhead significantly affects service performance.

The vtimer interrupt passthrough feature allows VM timer interrupts to be directly injected, eliminating VM entry and exit during injection for lower latency. This feature is applicable to the new Kunpeng 920 processor models and Kunpeng 950 processors.

>Due to hardware implementation, the active and pending states are not recorded in vtimer interrupt passthrough. If a VM depends on the active and pending states of timer interrupts for special processing, you can enable the vtimer interrupt status simulation in the libvirt configuration. The vtimer interrupt status simulation feature requires the support of the guest OS.

### Availability<a name="EN-US_TOPIC_0000002105901570"></a>

Version requirements:

openEuler 24.03 LTS SP4 or later is supported.

### Constraints<a name="EN-US_TOPIC_0000002070341631"></a>

- Only the new Kunpeng 920 processor models and Kunpeng 950 processors are supported.
- The active and pending states are not recorded in vtimer interrupt passthrough. If a VM depends on the active and pending states of timer interrupts for special processing, you can enable vtimer interrupt status simulation.
- The vtimer interrupt status simulation feature requires the support of the guest OS.

### Application Scenarios<a name="EN-US_TOPIC_0000002105901594"></a>

This feature is applicable to service scenarios that have stringent requirements on timer latency, such as high-precision timing tasks and real-time services. It avoids entry/exit operations during timer interrupt injection, thus reducing the injection latency and improving the performance of VM timer-related services.

## Feature Usage<a name="EN-US_TOPIC_0000002070341635"></a>

### Environment Requirements<a name="EN-US_TOPIC_0000002154855906"></a>

This document provides guidance based on the openEuler OS. Before performing operations, ensure that your hardware and software meet the requirements.

**Hardware Requirements<a name="section26241128"></a>**

[**Table 1**](#hardware-requirements) lists the hardware requirements.

**Table 1** Hardware requirements<a id="hardware-requirements"></a>

|Item|Description|
|--|--|
|Processor|New Kunpeng 920 processor model or Kunpeng 950 processor|
|BIOS configuration|Enabling GICv4.1 or GICv4.2|

**OS and Software Requirements<a name="section153345522324"></a>**

[**Table 2**](#os-and-software-requirements) lists the OS and software requirements.

**Table 2** OS and software requirements<a id="os-and-software-requirements"></a>

|Item|Version|How to Obtain|
|--|--|--|
|Host OS|openEuler 24.03 LTS SP4 or later|Install it using Yum.|
|Guest OS|openEuler 24.03 LTS SP4 or later (required when vtimer interrupt status simulation is enabled)|Install it using Yum.|
|libvirt|9.10.0 or later|Install it using Yum.|
|QEMU|8.2.0 or later|Install it using Yum.|

### Feature Enablement<a name="EN-US_TOPIC_0000002105901586"></a>

#### Enabling vtimer Interrupt Passthrough<a name="section_vtimer_enable"></a>

1. Add the following parameter to the end of the `linux` line of the corresponding kernel in the system boot configuration file `grub.cfg` of the host OS (for example, the file is in the `/boot/efi/EFI/openEuler/grub.cfg` directory for openEuler) to enable vtimer interrupt passthrough. Then, reboot the host OS to make the modification effective.

    ```shell
    kvm-arm.vtimer_irqbypass=1
    ```

2. Reboot the host OS for the configuration to take effect.

    ```shell
    reboot
    ```

#### Enabling the vtimer Interrupt Status Simulation<a name="section_vtimer_status_enable"></a>

>![](public_sys-resources/icon-notice.gif) **NOTICE:**
>The vtimer interrupt status simulation feature requires the support of the guest OS. Perform the following configurations only after you have confirmed that the guest OS supports this feature.

If a VM depends on the active and pending states of timer interrupts for special processing, add the following configuration to the libvirt XML VM configuration file:

```xml
<features>
  <kvm-vtimer-status enabled='yes'/>
</features>
```

After the configuration is complete, redefine the VM for the configuration to take effect.

```shell
virsh define vm.xml
```

### Feature Verification<a name="EN-US_TOPIC_0000002070181859"></a>

#### Verifying vtimer Interrupt Passthrough

Run the following command on the host OS to check whether the vtimer interrupt passthrough feature is enabled:

```shell
cat /sys/module/kvm_arm/parameters/vtimer_irqbypass
```

If the command output is `1`, the vtimer interrupt passthrough feature is enabled.

#### Verifying vtimer Interrupt Status Simulation

Run the following command on the VM to check whether the vtimer interrupt status simulation feature takes effect:

```shell
dmesg | grep vtimer
```

If the command output contains `using pvtimer status`, the feature has taken effect.

## Acronyms and Abbreviations<a name="EN-US_TOPIC_0000002106021550"></a>

|**Acronym/Abbreviation**|**Full Spelling**|
|--|--|
|VMM|virtual machine monitor|
|vtimer|virtual timer|
