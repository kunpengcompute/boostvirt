# BoostVirt Release Notes
## 2026-03-30
### Change History
| Issue| Release Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-03-30 | Added the NRI mode and a cluster affinity policy to the Kunpeng Topology Affinity Plugin (Kunpeng TAP).<br>Released the KAE-enabled Envoy feature to improve the gateway encryption and decryption performance.<br>Released the solution to handling single-core and single-page exceptions in VM scenarios.<br>Released the GICv4.1 overcommitment optimization solution to improve VM service performance in GICv4.1 overcommitment scenarios.|
### Version Requirements
#### Product Version
| Product Name     | Version     |
| --------- | -------- |
| BoostVirt | 26.0.RC1 |
#### Software Versions
| Feature         | Software Type                                                             | Version                                                                                                                                 |
| ------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Kunpeng TAP     | OS<br>Golang<br>Kubernetes<br>Docker<br>containerd<br>Kunpeng TAP | openEuler 20.03 LTS SP3/openEuler 22.03 LTS SP4/openEuler 24.03 LTS SP3<br>1.25<br>1.23.6/1.25.16<br>1.6.8/1.7.0<br>release-0.3 |
| KAE-enabled Envoy   | OS <br>Docker <br>KAE                                             | openEuler 24.03 LTS SP2 <br>20.10.13 or later, with the Docker Compose function supported<br>2.0                                                                 |
| VM single-core and single-page exception handling  | OS<br>libvirt<br>QEMU<br>Nginx<br>wrk                            | openEuler 24.03 LTS SP3<br>9.10.0-26.oe2403sp3 or later<br>8.2.0<br>1.24.0<br>4.2.0                                                     |
| GICv4.1 overcommitment optimization| OS<br>Kernel<br>libvirt<br>QEMU                                   | openEuler 24.03 LTS SP3<br>OLK 6.6<br>9.10.0<br>8.2.0                                                                               |
#### Hardware Versions
| Feature         | Item         | Description                                     |
| ------------- | ------------- | ----------------------------------------- |
| Kunpeng TAP     | Processor          | Kunpeng 920 series<br>Kunpeng 950                   |
| KAE-enabled Envoy   | Processor          | Kunpeng 920 series<br>Kunpeng 950                   |
| VM single-core and single-page exception handling  | Processor<br><br>Firmware| New Kunpeng 920 processor model<br>Kunpeng 950<br>Basic computing unit (BCU) CPLD: later than 7.0.0|
| GICv4.1 overcommitment optimization| Processor          | New Kunpeng 920 processor model                              |
#### Virus Scan Results
Virus scanning is not involved because no software package is released.
### Important Notes
None
### Release Notes
#### Change Description
##### Kunpeng TAP

**New functions:**

- Added a new cluster affinity policy. The cluster affinity policy is implemented by extending the existing topology-aware framework. It introduces cluster-level topology awareness to achieve topology-based container resource allocation. In addition, the Kunpeng processor model can be detected to automatically select the optimal affinity policy.
- Added the NRI mode. Kunpeng TAP can seamlessly integrate with containerd v1.7.0 or later through the standard NRI interface. This mode replaces the conventional proxy mode and eliminates the need to modify container runtime configurations.
##### KAE-enabled Envoy
Released the KAE-enabled Envoy feature in Kunpeng cloud native scenarios. This feature introduces a KAE private key provider to offload time-consuming encryption and decryption operations from the CPU to KAE. This accelerates encryption and decryption while releasing CPU computing power for other service workloads.
##### VM single-core and single-page exception handling
Released the VM single-core and single-page exception handling feature. With this feature enabled, single-core corrected errors (CEs) can be isolated online on Kunpeng servers without affecting service running; uncorrected errors (UEs) in a single page of memory affect only one process in a VM, preventing the VM from going offline.
##### GICv4.1 overcommitment optimization
Released the GICv4.1 overcommitment optimization feature for VM scenarios. This solution allows the GIC to skip VMOVP instructions when vCPUs are migrated between CPUs that share the same virtual processing element (vPE) table, thus improving VM service performance in overcommitment scenarios.
#### Resolved Issues
None
#### Known Issues
None
### Related Documentation
