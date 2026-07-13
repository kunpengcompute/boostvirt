# Release Notes

## 2026-06-30

### Change History

| Version| Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-06-30 |- Released Kunpeng QoS Controller developed through the reconstruction of the Kubernetes MPAM plugin. It supports MPAM memory access isolation and control, and is added with the SMT QoS interface.<br>- Released the performance optimization solution based on memory configuration reduction on Kunpeng servers in virtualization scenarios. The solution supports ZRAM-based memory compression, huge-page memory reclamation, and cold and hot page swapping, ensuring that the performance deterioration is less than 15% when memory overcommitment is enabled.<br>- Released the IPI latency optimization feature for high-specification VMs. This feature optimizes the SGI distribution path and VM cache to reduce the IPI latency of high-specification VMs and improve the VM multi-core communication performance.<br>- Released the Kunpeng adaptation guides for the AI sandbox solutions Kata and English2Bits (E2B). The Kata and E2B sandboxes can be deployed on Kunpeng servers.|

### Version Mapping

#### Product Version Information

| Product Name     | Version    |
| --------- | -------- |
| BoostVirt | 26.1.RC1 |

#### Software Version Mapping

| Feature         | Software         | Version                                     |
| ------------- | ------------- | ----------------------------------------- |
| Kunpeng QoS Controller     | - OS <br>- Kubernetes <br>- containerd          | - openEuler 24.03 LTS SP4 <br>- 1.28.*X*<br>- 1.7.*X*                   |
| Huge-page memory configuration reduction  | - OS<br>- libvirt<br>- QEMU | - openEuler 24.03 LTS SP3 (OLK 6.6 kernel)<br>- 9.10.0 <br>- 8.2.0 |
| IPI latency optimization for high-specification VMs| - OS<br>- libvirt<br>- QEMU | - openEuler 24.03 LTS SP3 (OLK 6.6 kernel 6.6.0-135)<br>- 9.10.0 <br>- 8.2.0 |
| Multi-ITS load balancing   | - OS<br>- libvirt<br>- QEMU<br>- BIOS | - openEuler 24.03 LTS SP3 (OLK 6.6 kernel 6.6.0-133)<br>- 9.10.0 <br>- 8.2.0<br> - 10.79 |

#### Hardware Version Mapping

| Feature         | Item         | Requirement                                     |
| ------------- | ------------- | ----------------------------------------- |
| Kunpeng QoS Controller     | Processor          | - Kunpeng 920 series<br>- Kunpeng 950                   |
| Huge-page memory configuration reduction  | Processor| New Kunpeng 920 processor model<br>|
| IPI latency optimization for high-specification VMs| Processor          | - Kunpeng 920 series<br>- Kunpeng 950                            |
| Multi-ITS load balancing   | Processor          | Kunpeng 950 processor                   |

#### Virus Scan Results

Virus scanning is not involved because no software package is released.

### Important Notes

None

### Release Notes

#### Change Description

##### Kunpeng QoS Controller

**New functions:**

- Through the reconstruction of the MPAM plugin, Kunpeng QoS Controller is compatible with the memory access isolation and control functions.
- The SMT QoS interface is added. After this interface is configured and enabled, idle instructions can be inserted to offline services through xint interrupts, and pipeline resources can be released to online services, reducing the interference of offline services to online services.

##### Huge-Page Memory Configuration Reduction

The huge-page memory configuration reduction feature is released. This feature uses the ZRAM module to compress and store the cold-page memory of VMs, and uses the Kunpeng Accelerator Engine (KAE) to improve the compression efficiency. In addition, this feature allows huge-page memory to be swappable. This implements efficient utilization of memory resources, so that more VMs can be deployed with limited physical memory.

##### IPI Latency Optimization for High-Specification VMs

The IPI latency optimization feature for high-specification VMs is released.

- This feature pre-calculates a compressed mapping table (`kvm_mpidr_data`) from MPIDR values to vCPU indexes when a VM runs for the first time. This feature transforms SGI distribution path traversal (O(n)) to direct search (O(1)), significantly reducing the SGI delivery latency.
- After the VM cache information is set, the detailed cache information can be injected into the VM so that the VM OS kernel can correctly identify the core cluster level (CCL) scheduling domain, improving the kernel scheduling efficiency.

##### Multi-ITS Load Balancing

The multi-ITS load balancing feature is released. On a server platform that supports multiple interrupt translation services (ITSs), virtual functions (VFs) created under a physical function (PF) of a physical peripheral component interconnect (PCI) device inherit the I/O Remapping Table (IORT)/PCI device interrupt domain parsing path by default. This default path forces VFs to select the same ITS as the PF. When the ITS used by the PF is overloaded, the VF performance is affected. This feature allows a PF to be configured with ITSs that VFs can use. The kernel can then determine the ITSs used by VFs based on the ITS used by the PF, platform topology rules, and chip capability trustlist, achieving ITS load balancing.

#### Resolved Issues

None

#### Known Issues

None

### Related Documentation

|Document|Description|Delivery Method|
|--|--|--|
|*Kunpeng QoS Controller User Guide*|Describes how to compile, deploy, and use Kunpeng QoS Controller.|Open-source repository|
|*VM Huge-Page Memory Configuration Reduction Feature Guide*|Describes how to deploy and use the VM huge-page memory configuration reduction feature on Kunpeng servers.|Open-source repository|
|*SGI Injection Affinity Optimization User Guide*<br> *VM Cache Information Configuration User Guide*|Describes the environment requirements and provides guidance on enabling the two core technologies of the IPI latency optimization solution for high-specification VMs.|Open-source repository|
|*VF ITS Selection User Guide*|Describes the environment requirements and provides guidance on enabling the multi-ITS load balancing feature.|Open-source repository|
|*E2B Deployment Guide*|Provides guidance on deploying the E2B sandbox on Kunpeng servers.|Open-source repository|
|*Kata Deployment Guide*|Provides guidance on deploying the Kata sandbox on Kunpeng servers.|Open-source repository|

## 2026-03-30

### Change History

| Version| Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-03-30 |- Added the NRI mode and a cluster affinity policy to the Kunpeng Topology Affinity Plugin (Kunpeng TAP).<br>- Released the KAE-enabled Envoy feature to improve the gateway encryption and decryption performance.<br>- Released the solution to handling single-core and single-page exceptions in VM scenarios.<br>- Released the GICv4.1 overcommitment optimization solution to improve VM service performance in GICv4.1 overcommitment scenarios.|

### Version Mapping

#### Product Version Information

| Product Name     | Version    |
| --------- | -------- |
| BoostVirt | 26.0.RC1 |

#### Software Version Mapping

| Feature         | Software                                                             | Version                                                                                                                                 |
| ------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Kunpeng TAP     | - OS<br>- Golang<br>- Kubernetes<br>- Docker<br>- containerd<br>- Kunpeng TAP | - openEuler 20.03 LTS SP3/openEuler 22.03 LTS SP4/openEuler 24.03 LTS SP3<br>- 1.25<br>- 1.23.6/1.25.16<br>- 20.10.14<br>- 1.6.8/1.7.0<br>- release-0.3 |
| KAE-enabled Envoy   | - OS <br>- Docker <br>- KAE                                             | - openEuler 24.03 LTS SP2 <br>- 20.10.13 or later, with the Docker Compose function supported<br>- 2.0                                                                 |
| VM single-core and single-page exception handling  | - OS<br>- libvirt<br>- QEMU<br>- Nginx<br>- wrk                            | - openEuler 24.03 LTS SP3<br>- 9.10.0-26.oe2403sp3 or later<br>- 8.2.0<br>- 1.24.0<br>- 4.2.0                                                     |
| GICv4.1 overcommitment optimization| - OS<br>- Kernel<br>- libvirt<br>- QEMU                                   | - openEuler 24.03 LTS SP3<br>- OLK 6.6<br>- 9.10.0<br>- 8.2.0                                                                               |

#### Hardware Version Mapping

| Feature         | Item         | Requirement                                     |
| ------------- | ------------- | ----------------------------------------- |
| Kunpeng TAP     | Processor          | - Kunpeng 920 series<br>- Kunpeng 950                   |
| KAE-enabled Envoy   | Processor          | - Kunpeng 920 series<br>- Kunpeng 950                   |
| VM single-core and single-page exception handling  | - Processor<br>- Firmware| - New Kunpeng 920 processor model<br>- Kunpeng 950<br>- Basic computing unit (BCU) CPLD: later than 7.0.0|
| GICv4.1 overcommitment optimization| Processor          | New Kunpeng 920 processor model                              |

#### Virus Scan Results

Virus scanning is not involved because no software package is released.

### Important Notes

None

### Release Notes

#### Change Description

##### Kunpeng TAP

**New functions:**

- A new cluster affinity policy is added. The cluster affinity policy is implemented by extending the existing topology-aware framework. It introduces cluster-level topology awareness to achieve topology-based container resource allocation. In addition, the Kunpeng processor model can be detected to automatically select the optimal affinity policy.
- The NRI mode is added. Kunpeng TAP can seamlessly integrate with containerd v1.7.0 or later through the standard NRI interface. This mode replaces the conventional proxy mode and eliminates the need to modify container runtime configurations.

##### KAE-enabled Envoy

The KAE-enabled Envoy feature in Kunpeng cloud native scenarios is released. This feature introduces a KAE private key provider to offload time-consuming encryption and decryption operations from the CPU to KAE. This accelerates encryption and decryption while releasing CPU computing power for other service workloads.

##### VM single-core and single-page exception handling

The VM single-core and single-page exception handling feature is released. With this feature enabled, single-core corrected errors (CEs) can be isolated online on Kunpeng servers without affecting service running; uncorrected errors (UEs) in a single page of memory affect only one process in a VM, preventing the VM from going offline.

##### GICv4.1 overcommitment optimization

The GICv4.1 overcommitment optimization feature for Kunpeng VM scenarios is released. This solution allows the GIC to skip VMOVP instructions when vCPUs are migrated between CPUs that share the same virtual processing element (vPE) table, thus improving VM service performance in overcommitment scenarios.

#### Resolved Issues

None

#### Known Issues

None

### Related Documentation

|Document|Description|Delivery Method|
|--|--|--|
|*Kunpeng Topology Affinity Plugin User Guide*|Describes how to compile, deploy, and use the Kubernetes topology affinity plugin.|Open-source repository|
|*KAE-enabled Envoy User Guide*|Provides operation instructions for KAE-enabled Envoy.|Open-source repository|
|*VM Single-Core and Single-Page Exception Handling Feature Guide*|Describes the environment requirements and provides guidance on enabling the VM single-core and single-page exception handling feature.|Open-source repository|
|*GICv4.1 Overcommitment Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the GIC overcommitment optimization feature.|Open-source repository|

### Obtaining Documentation<a name="EN-US_TOPIC_0000002544372643"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/boostvirt/tree/master/docs/en) to view or download required documents.
