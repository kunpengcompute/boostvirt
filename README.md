# 变更通知

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
|            | 拓扑感知调度            | 拓扑感知调度通过软硬协同，将CPU拓扑结构直通到虚拟机，Guest OS能进行Cluster级别调度优化，加速多线程调用效率。并基于CPU拓扑，通过超线程调度和动态绑核，尽可能减少vCPU跨cluster，提升虚机线性度。优化抢占过程的锁机制，提升超分场景性能。还支持硬件死锁检测，避免虚拟机卡死。 |
| 虚拟化IO加速    | 硬件加速器             | 硬件加速器介绍在虚拟机中使用鲲鹏加速引擎KAE（Kunpeng Accelerator Engine），以及进行vKAE热迁移的相关能力。                                                                                 |
| 虚拟化管理优化    | 热迁移KAE加速          | KAE的压缩模块提供了zlib标准接口KAEZlib，使用鲲鹏硬加速模块实现deflate算法，结合无损用户态驱动框架。因此KAE加速引擎可以替代原生zlib库加速虚拟机热迁移。                                                             |
|            | 热插拔               | 热插拔指虚拟机vCPU和内存动态调整的能力。vCPU热插拔通过中断和处理函数方式，动态模拟CPU上下电。Qemu虚拟机内存热插支持虚拟机包含初始内存为0的NUMA节点，并动态地向NUMA节点增加内存。                                                  |
|            | MPAM              | 在libvirt上使能MPAM特性，通过XML配置方式限制虚拟机的资源使用。                                                                                                                |
| 云原生基础架构    | KAE Device Plugin | 使用标准的K8s Device Plugin接口，易部署；充分利用KAE设备算力加速容器场景的加解密和压缩解压缩性能                                                                                            |
| 云原生资源亲和与隔离 | K8s 拓扑亲和插件        | 在Kubernetes容器超分场景下通过动态感知CPU、内存等资源的物理分布特征，确保关键工作负载能够在最优的NUMA节点上运行，显著减少跨节点访问带来的性能损耗。                                                                    |
|            | K8s MPAM插件        | 基于MPAM特性实现资源隔离，通过限制离线业务对内存带宽和L3缓存容量的占用，形成缓存/内存带宽的多层次资源隔离机制，有效解决云原生环境下混合部署时的资源争抢问题。                                                                    |
| 云原生高性能网络   | SR-IOV直通插件        | 通过K8s的Devices Plugin机制自动管理和直通SR-IOV设备                                                                                                                 |

# 贡献、建议与交流<a name="ZH-CN_TOPIC_0000002503696628"></a>

欢迎大家为社区做贡献，如果使用过程中有任何问题/建议，或者需要反馈特性需求和bug报告，可以提交[Issues](https://gitcode.com/boostkit/community/blob/master/docs/contributor/issue-submit.md)联系我们，具体贡献方法可参考[这里](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md)。同时也欢迎大家在[讨论专区](https://gitcode.com/boostkit/community/discussions)展开讨论交流。感谢您的支持。

# LICENSE<a name="ZH-CN_TOPIC_0000002535536615"></a>

本项目采用Apache License 2.0许可证。详见[LICENSE](https://gitcode.com/boostkit/cloud-virtual/blob/master/LICENSE)文件。
本项目的文档适用CC-BY 4.0许可证，具体请参见[LICENSE](https://gitcode.com/boostkit/boostvirt/blob/master/docs/zh/LICENSE)文件。
