# Change Notification

<!-- md-trans-meta sourceCommit=018b81addca2fcd991de9b1f328f6c98174d7aa1 translatedAt=2026-10-16T07:09:48.058Z pushedAt=2026-10-19T03:10:11.338Z -->

- [2026-09-30]: Updated the information about BoostVirt features released on September 30.<br> - Released the kunpeng-perf-monitor plugin, which supports metric collection and visualization on Kunpeng 950 processors.<br> - Released the Kunpeng-TAP Pcore Binding plugin, which supports containers binding only to physical cores.<br> - Added official support for CubeSandbox deployment on Kunpeng and provided tuning guidance.

- [2026-06-30]: Updated the information about BoostVirt features released on June 30.<br> - Released Kunpeng QoS Controller developed through the refactoring of the Kubernetes MPAM plugin. It supports MPAM memory access isolation and control, and adds the SMT QoS interface.<br>- Released the performance optimization solution based on memory configuration reduction on Kunpeng servers in virtualization scenarios. The solution supports ZRAM-based memory compression, huge-page memory reclamation, and cold and hot page swapping, ensuring that the performance degradation is less than 15% when memory overcommitment is enabled.<br>- Released the IPI latency optimization feature for high-specification VMs. This feature optimizes the SGI distribution path and VM cache to reduce the IPI latency of high-specification VMs and improve the multi-core communication performance of VMs.<br>- Released the Kunpeng adaptation guides for the AI sandbox solutions Kata and English2Bits (E2B). The Kata and E2B sandboxes can be deployed on Kunpeng servers.
- [2026-03-24]: Updated the overview of the BoostVirt SIG open-source projects.

# Project Introduction<a name="EN-US_TOPIC_0000002535416437"></a>

## Overview

BoostVirt is a virtualization acceleration suite launched by the Kunpeng BoostKit cloud computing team. It aims to use Kunpeng hardware to accelerate cloud-native and virtualized applications, maximize computing performance, minimize network and storage I/O overhead, and optimize workload efficiency within VMs and containers, while providing open-source enablement for virtualization.

## Community Homepage

[BoostVirt homepage in the Kunpeng community](https://www.hikunpeng.com/boostkit/boost-x/virt)

## Open-Source Project Overview

| Code Repository               | Description                                                                            | Address                                     |
| ------------------ | --------------------------------------------------------------------------------- | ------------------------------------------ |
| cloud-native       | This repository is a collection of Kunpeng cloud-native projects, including multiple cloud-native components optimized for Kunpeng processors. Based on Kubernetes, it provides cloud-native infrastructure and network and management plugins optimized for Kunpeng.               | [Link](https://gitcode.com/boostkit/cloud-native)  |
| cloud-virtual      | This repository is a collection of Kunpeng virtualization projects, including multiple virtualization components optimized for Kunpeng processors. It covers Kunpeng-affinity optimization features for core virtualization and I/O software stacks such as KVM, QEMU, libvirt, DPDK, and SPDK.| [Link](https://gitcode.com/boostkit/cloud-virtual)|

# Feature Description<a name="EN-US_TOPIC_0000002535536613"></a>

| Category      | Feature             | Description                                                                                                                                                 |
| ---------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Virtualized CPU acceleration  | Interrupt passthrough             | Enables GICv4.1 direct virtual interrupt injection and direct vSGI injection on the new Kunpeng 920 processor model to significantly reduce interrupt response latency and boost throughput for network and I/O-intensive workloads.                                                                    |
|            | GICv4.1 overcommitment optimization         | Allows the GIC to skip VMOVP instructions when vCPUs are migrated between CPUs that share the same virtual processing element (vPE) table, thus improving VM service performance in overcommitment scenarios.  |
|            | IPI latency optimization for high-specification VMs    | By optimizing the scheduling on the SGI distribution path to O(1), significantly reduces the end-to-end IPI latency for high-specification VMs, improving service performance in multi-core concurrency scenarios.  |
|            | Topology-aware scheduling            | Leverages hardware-software synergy to pass the CPU topology through to the VMs, allowing the guest OS to perform cluster-level scheduling optimizations and accelerate multi-threaded execution. Hyper-threading scheduling and dynamic core binding reduce vCPU cross-cluster overhead based on the CPU topology to improve VM linearity. Optimized preemption locking minimizes contention and boosts overall performance in overcommitment environments. Hardware deadlock detection is also supported to prevent VMs from hanging. |
|            | VM cache topology information passthrough   | Allows the L3 cache to be manually attached at the CCL level to keep the MC scheduling domain within a single CCL, reducing cross-CCL task scheduling and IPI communication and improving memory access locality.|
| Virtualized I/O acceleration    | Hardware accelerator             | Empowers VMs with hardware acceleration through KAE, including support for live migration.                                                                                 |
|                  | VF ITS selection          | Provides a PF-level sysfs interface, which allows users to configure a group of ITSs that can be used by VFs for a PF (by specifying the ITS raw indexes). Before accepting the configuration, the kernel verifies the configuration based on the ITS currently used by the PF, platform topology rules, and the chip capability trustlist. The configuration is saved and takes effect during subsequent VF enumeration only when all conditions are met.        |
| Virtualization management optimization    | KAE-accelerated live migration          | Provides a standard zlib interface (KAEZlib). It uses the Kunpeng hardware acceleration module to implement the Deflate algorithm and works with the lossless user-mode driver framework. Therefore, KAE can replace the native zlib library to accelerate VM live migration.                                                             |
|            | Hot-plug              | Refers to the capability of dynamically adjusting vCPUs and memory of VMs. vCPU hot-plug dynamically simulates CPU power-on and power-off through interrupt mechanisms and processing functions. It enables QEMU VMs to include NUMA nodes with zero initial memory and dynamically allocate memory to them.                                                 |
|            | MPAM              | Enables the MPAM feature on libvirt and restricts the resource usage of VMs through XML configuration.                                                                                                                |
|               | VM single-core and single-page exception handling | Allows single-core corrected errors (CEs) to be isolated online on Kunpeng servers without affecting service running; enables uncorrected errors (UEs) in a single page of memory to affect only one process in a VM, preventing the VM from going offline. |
| Virtualized memory optimization | Huge-page memory configuration reduction | Uses a PV mechanism to reclaim idle page cache and perform memory defragmentation within VMs, the ZRAM module to compress and store the cold-page memory of VMs, and KAE to improve the compression efficiency. In addition, this feature allows huge-page memory to be swappable. This enables efficient utilization of memory resources, so that more VMs can be deployed with limited physical memory. |
| Cloud-native infrastructure   | KAE device plugin | Simplifies deployment through the standard Kubernetes Device Plugin interface while boosting encryption, decryption, compression, and decompression performance in containerized scenarios.                                                                                            |
|       | kunpeng-perf-monitor plugin | Serves as a Prometheus Exporter for Kunpeng servers. This plugin is deployed in DaemonSet mode on each node in a Kubernetes cluster. It supports the collection of node-level hardware and kernel performance metrics and can be connected to Prometheus through configuration to implement visualized monitoring of related metrics. |
| Cloud-native resource affinity and isolation| Kubernetes topology affinity plugin       | In Kubernetes overcommitment scenarios, the plugin dynamically senses the physical distribution of CPU and memory resources. By ensuring critical workloads run on the optimal NUMA nodes, it minimizes performance loss caused by cross-node access.                                                                    |
|  | Kunpeng-TAP Pcore Binding plugin | Supports physical core binding, instead of hyper-threading binding, based on the physical machine topology of Kunpeng servers. |
|            | Kubernetes MPAM plugin       | Eliminates resource contention in hybrid deployments with MPAM-based resource isolation. By intelligently partitioning L3 cache and memory bandwidth, it protects critical services from interference by offline workloads.                                                                    |
| Cloud-native high-performance network  | SR-IOV device plugin       | Automatically manages and implements passthrough for SR-IOV devices using the Kubernetes device plugin mechanism.                                                                                                                 |

# Contributions, Suggestions, and Discussions<a name="EN-US_TOPIC_0000002503696628"></a>

We welcome your contributions to the community. If you have any questions/suggestions or want to provide feedback on feature requirements and bug reports, you can [submit issues](https://gitcode.com/boostkit/community/blob/master/docs/contributor/issue-submit.md). For details, see the [contribution guideline](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md). You are also welcome to share insights in the [Discussions](https://gitcode.com/boostkit/community/discussions). Thank you for your support.

# License<a name="EN-US_TOPIC_0000002535536615"></a>

This project is licensed under Apache License 2.0. For details, see [LICENSE](https://gitcode.com/boostkit/cloud-virtual/blob/master/LICENSE).
The documents of this project are licensed under CC-BY 4.0. For details, see [LICENSE](https://gitcode.com/boostkit/boostvirt/blob/master/docs/en/LICENSE).
