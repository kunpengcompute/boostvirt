# PMU虚拟化 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002278901521"></a>

本文主要介绍如何在使用openEuler操作系统的鲲鹏920系列处理器上部署和使能PMU虚拟化特性。

PMU（Performance Monitoring Unit，性能监控单元）是ARM处理器提供的一个硬件性能事件采集系统，用于记录和计数各种微架构事件。使能PMU虚拟化特性后，用户可以通过perf工具采集虚拟机内部的PMU事件，从而进行性能分析与调优。

**版本支持<a name="section152361333185213"></a>**

- 物理机支持openEuler 22.03 LTS SP4、openEuler 24.03 LTS SP1的操作系统版本。
- 虚拟机支持openEuler 22.03 LTS SP4、openEuler 24.03 LTS SP1的操作系统版本。
- License支持：无。

**应用场景<a name="section342793714538"></a>**

本特性能够实现在虚拟机内部采集PMU事件，有助于进行虚拟机OS与业务软件的性能分析与调优。


## 特性使用<a name="ZH-CN_TOPIC_0000002243915872"></a>

### 环境要求<a name="ZH-CN_TOPIC_0000002278874913"></a>

在使用特性前，请确认已满足软件与硬件要求。

**硬件要求<a name="zh-cn_topic_0000001217080138_section10273165810425"></a>**



**表 1** 硬件要求<a id="硬件要求"></a>

|项目|规格|
|--|--|
|处理器|鲲鹏920系列处理器|


**操作系统和软件要求<a name="section1240364411598"></a>**



**表 2** 操作系统与软件要求<a id="操作系统与软件要求"></a>

|软件名称|版本|获取方法|
|--|--|--|
|物理机与虚拟机操作系统|已经验证的版本：openEuler 22.03 LTS SP4openEuler 24.03 LTS SP1|openEuler 22.03 LTS SP4：[获取链接](https://mirrors.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP4/ISO/aarch64/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso)<br>openEuler 24.03 LTS SP1：[获取链接](https://mirrors.huaweicloud.com/openeuler/openEuler-24.03-LTS-SP1/ISO/aarch64/openEuler-24.03-LTS-SP1-everything-aarch64-dvd.iso)|
|QEMU|openEuler 22.03 LTS SP4：6.2.0openEuler 24.03 LTS SP1：8.2.0|通过配置Yum源安装|
|libvirt|openEuler 22.03 LTS SP4：6.2.0openEuler 24.03 LTS SP1：9.10.0|通过配置Yum源安装|



### 使能与验证<a name="ZH-CN_TOPIC_0000002243756076"></a>

#### 安装虚拟机运行环境<a name="ZH-CN_TOPIC_0000002278954429"></a>

>![](public_sys-resources/icon-note.gif) **说明：** 
>-   安装前需要自行完成Yum源的配置，以及准备虚拟机安装需要的系统镜像文件，安装过程中的路径信息根据实际情况进行修改。
>-   如虚拟机使用openEuler 24.03 LTS SP1系统，创建虚拟机时需要在命令末尾添加参数**--osinfo detect=on,require=off**。

1. 安装QEMU和libvirt。

    ```
    yum -y install qemu libvirt edk2-aarch64.noarch virt-install
    ```

2. 创建虚拟机，本文以创建20G的qcow2格式的磁盘镜像文件为例进行说明。

    ```
    virt-install --name=<vmname> --vcpus=32 --ram=65536 --disk path=/home/images/img.qcow2,format=qcow2,size=20,bus=virtio --location /home/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso --network network=default --nographics
    ```


#### 测试虚拟机PMU事件采集<a name="ZH-CN_TOPIC_0000002244115250"></a>

>![](public_sys-resources/icon-note.gif) **说明：** 
>安装perf工具前需要自行完成虚拟机内Yum源的配置。

1. 启动并进入虚拟机。

    ```
    virsh start <vmname> --console
    ```

2. 安装perf工具。

    ```
    yum -y install perf
    ```

3. 执行10秒虚拟机内核基础事件采集，得到如下图结果表示PMU虚拟化特性已经使能。

    ```
    perf stat -a sleep 10
    ```

    ![](figures/zh-cn_image_0000002279062345.png)




