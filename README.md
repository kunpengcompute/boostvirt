# 变更通知

- [2026.09.30]：更新BoostVirt 930发布特性信息<br> - 首次发布鲲鹏容器性能指标监控插件（Kunpeng Perf Monitor），支持鲲鹏950处理器指标采集和可视化。<br> - 首次发布鲲鹏物理拓扑绑核插件（Kunpeng TAP Pcore Binding）首个版本，支持容器只绑物理核。<br> - 鲲鹏正式支持CubeSandbox沙箱部署并提供优化指南。

- [2026.06.30]：更新BoostVirt 630发布特性信息<br> - 首次发布鲲鹏混部调优插件，重构K8s MPAM插件，支持MPAM访存隔离和控制功能，新增SMT Qos接口。<br>- 首次针对鲲鹏服务器内存减配虚拟化场景发布性能优化解决方案，通过支持ZRAM压缩内存、大页内存回收和冷热页交换实现内存超分时性能下降小于15%。<br>- 首次发布大规格虚拟机IPI时延优化特性，通过优化SGI分发路径和虚拟机缓存，从而降低大规格虚拟机的IPI时延，提高虚拟机多核通信性能。<br>- 首次针对AI沙箱方案Kata和E2B发布鲲鹏适配指导，支持在鲲鹏服务器上进行kata和E2B的沙箱部署。
- [2026.03.24]：更新BoostVirt SIG开源项目全景介绍

# 项目介绍<a name="ZH-CN_TOPIC_0000002535416437"></a>

## 概述

BoostVirt是鲲鹏BoostKit云计算团队推出的虚拟化加速套件，旨在基于鲲鹏硬件提供虚拟化和云原生的应用加速能力，聚焦提升鲲鹏虚拟化计算性能、降低网络存储的IO损耗、提升业务在虚拟机和容器中的性能，提供虚拟化开源使能优化。

## 社区主页

[鲲鹏社区BoostVirt主页](https://www.hikunpeng.com/boostkit/boost-x/virt)

## 开源项目全景

| 代码仓                | 代码仓介绍                                                                             | 代码仓地址                                      |
| ------------------ | --------------------------------------------------------------------------------- | ------------------------------------------ |
| cloud-native       | 鲲鹏云原生项目集合，包含多个针对鲲鹏处理器优化的云原生组件，基于Kubernetes提供云原生基础架构，网络和管理等鲲鹏亲和优化插件                |[获取链接](https://gitcode.com/boostkit/cloud-native)  |
| cloud-virtual      | 鲲鹏虚拟化项目集合，包含多个针对鲲鹏处理器优化的虚拟化组件，覆盖KVM、QEMU、Libvirt、DPDK、SPDK等核心虚拟化与I/O软件栈的鲲鹏亲和优化特性。 | [获取链接](https://gitcode.com/boostkit/cloud-virtual) |

# 特性介绍<a name="ZH-CN_TOPIC_0000002535536613"></a>

| 特性分类       | 特性名称              | 特性介绍                                                                                                                                                  |
| ---------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 虚拟化CPU加速   | 中断直通              | 中断直通特性通过在920新型号处理器使能GICv4.1的直接注入虚拟中断和直接注入vSGI的中断直通类型，可以显著降低中断响应时延，提升网络/IO密集型业务吞吐量。                                                                    |
|            | GICv4.1超分优化         | 通过支持vCPU在共享同一个vpe表的CPU间迁移的时候可以跳过VMOVP指令，从而提高超分场景下的虚拟机业务性能。  |
|            | 大规格虚机IPI时延优化    | 通过SGI分发路径的O(1)的调度优化，显著降低大规格虚拟机中IPI的端到端时延，提升多核并发场景下的业务性能。  |
|            | 拓扑感知调度            | 拓扑感知调度通过软硬协同，将CPU拓扑结构直通到虚拟机，Guest OS能进行Cluster级别调度优化，加速多线程调用效率。并基于CPU拓扑，通过超线程调度和动态绑核，尽可能减少vCPU跨cluster，提升虚机线性度。优化抢占过程的锁机制，提升超分场景性能。还支持硬件死锁检测，避免虚拟机卡死。 |
|            | 虚拟机缓存拓扑信息透传   | 支持手动设置L3 cache挂载在CCL层级，控制MC调度域的范围在一个CCL内，减少跨CCL的任务调度与IPI通信，提升访存局部性。|
| 虚拟化IO加速    | 硬件加速器             | 硬件加速器介绍在虚拟机中使用鲲鹏加速引擎KAE（Kunpeng Accelerator Engine），以及进行vKAE热迁移的相关能力。                                                                                 |
|                  | VF设备选择ITS          | 提供PF级sysfs接口，允许用户为某个PF配置一组VF可使用的ITS设备（通过指定ITS raw index）。内核在接受配置前会根据PF当前实际使用的ITS设备、平台拓扑规则和芯片能力白名单进行校验。只有全部条件满足时，配置才会被保存并在后续VF枚举时生效。        |
| 虚拟化管理优化    | 热迁移KAE加速          | KAE的压缩模块提供了zlib标准接口KAEZlib，使用鲲鹏硬加速模块实现deflate算法，结合无损用户态驱动框架。因此KAE加速引擎可以替代原生zlib库加速虚拟机热迁移。                                                             |
|            | 热插拔               | 热插拔指虚拟机vCPU和内存动态调整的能力。vCPU热插拔通过中断和处理函数方式，动态模拟CPU上下电。Qemu虚拟机内存热插支持虚拟机包含初始内存为0的NUMA节点，并动态地向NUMA节点增加内存。                                                  |
|            | MPAM              | 在libvirt上使能MPAM特性，通过XML配置方式限制虚拟机的资源使用。                                                                                                                |
|               | 虚拟机单核单页异常处理 | 在鲲鹏系列服务器上实现单核CE（Corrected Error）故障在线隔离，不影响业务的运行；单页内存UE（Uncorrected Error）故障只影响虚拟机内一个进程，避免虚拟机直接下线。 |
| 虚拟化内存优化 | 虚拟机大页内存减配优化 | 通过PV的方式实现虚拟机内部空闲pagecache回收以及内存碎片整理，并且利用ZRAM模块对虚拟机冷页内存进行压缩存储，结合KAE硬件加速技术提升压缩效率，同时解决大页内存不支持swap的问题，实现内存资源的高效利用，从而在有限物理内存下部署更多虚拟机。 |
| 云原生基础架构    | KAE Device Plugin | 使用标准的K8s Device Plugin接口，易部署；充分利用KAE设备算力加速容器场景的加解密和压缩解压缩性能                                                                                            |
|       | 鲲鹏性能指标监控插件 | 作为面向鲲鹏服务器的Prometheus Exporter，以DaemonSet方式部署在Kubernetes集群每个节点上，能够支持采集节点级的硬件与内核性能指标并支持配置接入Prometheus，实现相关指标的可视化检测 |
| 云原生资源亲和与隔离 | K8s 拓扑亲和插件        | 在Kubernetes容器超分场景下通过动态感知CPU、内存等资源的物理分布特征，确保关键工作负载能够在最优的NUMA节点上运行，显著减少跨节点访问带来的性能损耗。                                                                    |
|  | 鲲鹏物理拓扑绑核插件 | 支持根据鲲鹏服务器物理机拓扑进行物理核绑核而非超线程绑核 |
|            | K8s MPAM插件        | 基于MPAM特性实现资源隔离，通过限制离线业务对内存带宽和L3缓存容量的占用，形成缓存/内存带宽的多层次资源隔离机制，有效解决云原生环境下混合部署时的资源争抢问题。                                                                    |
| 云原生高性能网络   | SR-IOV直通插件        | 通过K8s的Devices Plugin机制自动管理和直通SR-IOV设备                                                                                                                 |

# 贡献、建议与交流<a name="ZH-CN_TOPIC_0000002503696628"></a>

欢迎大家为社区做贡献，如果使用过程中有任何问题/建议，或者需要反馈特性需求和bug报告，可以提交[Issues](https://gitcode.com/boostkit/community/blob/master/docs/contributor/issue-submit.md)联系我们，具体贡献方法可参考[这里](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md)。同时也欢迎大家在[讨论专区](https://gitcode.com/boostkit/community/discussions)展开讨论交流。感谢您的支持。

# LICENSE<a name="ZH-CN_TOPIC_0000002535536615"></a>

本项目采用Apache License 2.0许可证。详见[LICENSE](https://gitcode.com/boostkit/cloud-virtual/blob/master/LICENSE)文件。
本项目的文档适用CC-BY 4.0许可证，具体请参见[LICENSE](https://gitcode.com/boostkit/boostvirt/blob/master/docs/zh/LICENSE)文件。
