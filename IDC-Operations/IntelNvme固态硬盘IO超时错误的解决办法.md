# 关于Intel NVMe协议固态硬盘IO Timeout错误的解决办法

## 硬件与错误环境:
> 硬盘
```
root@F1-32:~# lspci | grep -i Non-Volatile
04:00.0 Non-Volatile memory controller: Intel Corporation Express Flash NVMe P4500/P4600
06:00.0 Non-Volatile memory controller: Intel Corporation Express Flash NVMe P4500/P4600
07:00.0 Non-Volatile memory controller: Intel Corporation Express Flash NVMe P4500/P4600
08:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981
```

> 报错

```
root@F1-32:~# tail -n 10 /var/log/kern.log
Aug  3 18:39:41 F1-32 kernel: [982494.258887] Total swap = 205520892kB
Aug  3 18:39:41 F1-32 kernel: [982494.258887] 267353073 pages RAM
Aug  3 18:39:41 F1-32 kernel: [982494.258888] 0 pages HighMem/MovableOnly
Aug  3 18:39:41 F1-32 kernel: [982494.258888] 4188158 pages reserved
Aug  3 18:39:41 F1-32 kernel: [982494.258888] 0 pages cma reserved
Aug  3 18:39:41 F1-32 kernel: [982494.258888] 0 pages hwpoisoned
Aug  3 18:40:19 F1-32 kernel: [982532.705531] nvme nvme0: I/O 292 QID 11 timeout, completion polled
Aug  3 19:28:04 F1-32 kernel: [985396.809012] nvme nvme2: I/O 14 QID 51 timeout, completion polled
Aug  3 22:29:40 F1-32 kernel: [996293.127939] nvme nvme1: I/O 527 QID 49 timeout, completion polled
Aug  3 22:56:57 F1-32 kernel: [997930.457845] nvme nvme1: I/O 668 QID 14 timeout, completion polled
```

## 工具：

**IntelMAS**：Intel Memory and Storge

[Download Page](https://downloadcenter.intel.com/download/30556/Intel-Memory-and-Storage-Tool-CLI-Command-Line-Interface-)

## 解决过程：

0. 下载并解压
```
wget https://downloadmirror.intel.com/30556/eng/Intel%C2%AE_MAS_CLI_Tool_Linux_1.9.zip
mkdir intelmas
unzip Intel®_MAS_CLI_Tool_Linux_1.9.zip -d ./intelmas/
cd ./intelmas
dpkg -i intelmas_1.9.147-0_amd64.deb
```

> example output

```
root@F1-32:~/intelmas# dpkg -i intelmas_1.9.147-0_amd64.deb
Selecting previously unselected package intelmas.
(Reading database ... 164643 files and directories currently installed.)
Preparing to unpack intelmas_1.9.147-0_amd64.deb ...
Unpacking intelmas (1.9.147-0) ...
Setting up intelmas (1.9.147-0) ...
```

1. 查看系统内SSD简略信息总览

```
intelmas show -intelssd
```
> example output

```
root@F1-32:~/intelmas# intelmas show -intelssd

- 0 Intel SSD DC P4510 Series PHLJ030403H44P0DGN -

Bootloader : 0203
Capacity : 4000.79 GB
DevicePath : /dev/nvme0n1
DeviceStatus : Healthy
Firmware : VDV10131
FirmwareUpdateAvailable : Firmware=VDV10182 Bootloader=VB1B0181
Index : 0
MaximumLBA : 7814037167
ModelNumber : INTEL SSDPE2KX040T8
NamespaceId : 1
PercentOverProvisioned : 100.00
ProductFamily : Intel SSD DC P4510 Series
SMARTEnabled : True
SectorDataSize : 512
SerialNumber : PHLJ030403H44P0DGN

- 1 Intel SSD DC P4510 Series PHLJ029601SF4P0DGN -

Bootloader : 0203
Capacity : 4000.79 GB
DevicePath : /dev/nvme1n1
DeviceStatus : Healthy
Firmware : VDV10131
FirmwareUpdateAvailable : Firmware=VDV10182 Bootloader=VB1B0181
Index : 1
MaximumLBA : 7814037167
ModelNumber : INTEL SSDPE2KX040T8
NamespaceId : 1
PercentOverProvisioned : 100.00
ProductFamily : Intel SSD DC P4510 Series
SMARTEnabled : True
SectorDataSize : 512
SerialNumber : PHLJ029601SF4P0DGN

- 2 Intel SSD DC P4510 Series PHLJ030402HC4P0DGN -

Bootloader : 0203
Capacity : 4000.79 GB
DevicePath : /dev/nvme2n1
DeviceStatus : Healthy
Firmware : VDV10131
FirmwareUpdateAvailable : Firmware=VDV10182 Bootloader=VB1B0181
Index : 2
MaximumLBA : 7814037167
ModelNumber : INTEL SSDPE2KX040T8
NamespaceId : 1
PercentOverProvisioned : 100.00
ProductFamily : Intel SSD DC P4510 Series
SMARTEnabled : True
SectorDataSize : 512
SerialNumber : PHLJ030402HC4P0DGN

- 3 SSD S3W9NX0N703091 -

Capacity : 512.11 GB
DevicePath : /dev/nvme3n1
DeviceStatus : Unknown
Firmware : EXA7302Q
FirmwareUpdateAvailable : The selected drive contains current firmware as of this tool release.
Index : 3
MaximumLBA : 1000215215
ModelNumber : SAMSUNG MZVLB512HAJQ-00007
PercentOverProvisioned : Property not found
ProductFamily : Property not found
SMARTEnabled : Property not found
SectorDataSize : 512
SerialNumber : S3W9NX0N703091
```

确认硬盘固件版本号（Firmware）,Intel P4510 Serial 最新固件应该是`VDV10182`

2. 使用`intelmas`工具升级固态固件版本

```
intelmas load -intelssd {0 | 1 | 2 | 3}
最后的数字要依系统实际情况而定，上面的简略信息总览里面
- 2 Intel SSD DC P4510 Series PHLJ030402HC4P0DGN -
- 1 Intel SSD DC P4510 Series PHLJ029601SF4P0DGN -
```

> example output

```
root@F1-32:~/intelmas# intelmas load -intelssd 0
WARNING! You have selected to update the drives firmware!
Proceed with the update? (Y|N): y
Checking for firmware update...

- Intel SSD DC P4510 Series PHLJ030403H44P0DGN -

Status : Firmware updated successfully. Please reboot the system.
```

3. 确认固件版本号

```
继续使用ssd简略信息总览
intelmas show -intelssd
```

> example output

```
注意- 0 Intel SSD DC P4510 Series PHLJ030403H44P0DGN -的Firmware

root@F1-32:~/intelmas# intelmas show -intelssd

- 0 Intel SSD DC P4510 Series PHLJ030403H44P0DGN -

Bootloader : 0181
Capacity : 4000.79 GB
DevicePath : /dev/nvme0n1
DeviceStatus : Healthy
Firmware : VDV10182
FirmwareUpdateAvailable : The selected drive contains current firmware as of this tool release.
Index : 0
MaximumLBA : 7814037167
ModelNumber : INTEL SSDPE2KX040T8
NamespaceId : 1
PercentOverProvisioned : 100.00
ProductFamily : Intel SSD DC P4510 Series
SMARTEnabled : True
SectorDataSize : 512
SerialNumber : PHLJ030403H44P0DGN

- 1 Intel SSD DC P4510 Series PHLJ029601SF4P0DGN -

Bootloader : 0203
Capacity : 4000.79 GB
DevicePath : /dev/nvme1n1
DeviceStatus : Healthy
Firmware : VDV10131
FirmwareUpdateAvailable : Firmware=VDV10182 Bootloader=VB1B0181
Index : 1
MaximumLBA : 7814037167
ModelNumber : INTEL SSDPE2KX040T8
NamespaceId : 1
PercentOverProvisioned : 100.00
ProductFamily : Intel SSD DC P4510 Series
SMARTEnabled : True
SectorDataSize : 512
SerialNumber : PHLJ029601SF4P0DGN

- 2 Intel SSD DC P4510 Series PHLJ030402HC4P0DGN -

Bootloader : 0203
Capacity : 4000.79 GB
DevicePath : /dev/nvme2n1
DeviceStatus : Healthy
Firmware : VDV10131
FirmwareUpdateAvailable : Firmware=VDV10182 Bootloader=VB1B0181
Index : 2
MaximumLBA : 7814037167
ModelNumber : INTEL SSDPE2KX040T8
NamespaceId : 1
PercentOverProvisioned : 100.00
ProductFamily : Intel SSD DC P4510 Series
SMARTEnabled : True
SectorDataSize : 512
SerialNumber : PHLJ030402HC4P0DGN

- 3 SSD S3W9NX0N703091 -

Capacity : 512.11 GB
DevicePath : /dev/nvme3n1
DeviceStatus : Unknown
Firmware : EXA7302Q
FirmwareUpdateAvailable : The selected drive contains current firmware as of this tool release.
Index : 3
MaximumLBA : 1000215215
ModelNumber : SAMSUNG MZVLB512HAJQ-00007
PercentOverProvisioned : Property not found
ProductFamily : Property not found
SMARTEnabled : Property not found
SectorDataSize : 512
SerialNumber : S3W9NX0N703091
```