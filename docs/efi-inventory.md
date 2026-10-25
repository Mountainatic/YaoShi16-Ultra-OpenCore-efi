# EFI 与仓库文件清单

本文件按目录列出仓库中的全部文件、大小与对应版本。版本读取自各 kext 的 `Info.plist`（`CFBundleVersion`），启用状态读取自 `EFI/OC/config.plist` 的 `Kernel > Add`。

清单生成于配置脱敏与调试备份清理之后，所列内容与仓库当前状态一致。`docs/efi-inventory.md` 自身不计入下表。

## 顶层概览

| 路径 | 文件数 | 大小 | 内容 |
| --- | ---: | ---: | --- |
| `EFI/BOOT` | 1 | 53,248 B | EFI 启动入口 |
| `EFI/OC（根目录）` | 2 | 870,648 B | OpenCore.efi 与 config.plist |
| `EFI/OC/ACPI` | 11 | 601,383 B | ACPI 表 |
| `EFI/OC/Drivers` | 4 | 850,440 B | UEFI 驱动 |
| `EFI/OC/Kexts` | 74 | 56,940,159 B (54.30 MB) | 内核扩展（28 个 kext） |
| `EFI/OC/Resources` | 58 | 3,609,827 B (3.44 MB) | OpenCanopy 资源 |
| `docs` | 8 | 106,730 B | 文档与原机记录 |
| `README.md` | 1 | 22,212 B | 仓库说明 |
| `.gitignore` | 1 | 16 B | 忽略规则 |
| 仓库合计 | 160 | 63,054,844 B (60.13 MB) | 含全部文件 |

## EFI/BOOT

| 文件 | 大小 |
| --- | ---: |
| `BOOTx64.efi` | 53,248 B |

## EFI/OC 根目录

| 文件 | 大小 | 说明 |
| --- | ---: | --- |
| `OpenCore.efi` | 831,488 B | OpenCore 1.0.8 主体 |
| `config.plist` | 39,160 B | 脱敏后的配置模板 |

## EFI/OC/ACPI（11 张表）

| 文件 | 大小 | 作用 |
| --- | ---: | --- |
| `APIC-LAPIC.aml` | 658 B | 同机采集的 MADT，24 组处理器 UID/APIC 映射，x2APIC 条目以 Local APIC 形式呈现 |
| `DSDT.aml` | 596,487 B | 固件 DSDT，前向 Scope 声明交由 macOS ACPICA 解释 |
| `SSDT-EC.aml` | 125 B | 嵌入式控制器描述 |
| `SSDT-GPI0.aml` | 124 B | I2C GPIO 设备描述 |
| `SSDT-MCHC.aml` | 105 B | 内存控制器中枢描述 |
| `SSDT-PLUG-ALT.aml` | 3,112 B | 处理器电源管理注入（替代方案） |
| `SSDT-RTCAWAC.aml` | 78 B | RTC 与 AWAC 时钟设备 |
| `SSDT-SBUS.aml` | 128 B | SMBus 设备描述 |
| `SSDT-USB-Reset.aml` | 181 B | USB 复位属性 |
| `SSDT-USBX.aml` | 166 B | USB 供电属性 |
| `SSDT-XOSI.aml` | 219 B | `_OSI` 到 `XOSI` 改名的配合表 |

## EFI/OC/Drivers（4 个）

| 文件 | 大小 | 作用 |
| --- | ---: | --- |
| `HfsPlus.efi` | 37,892 B | Apple HFS+ 文件系统驱动 |
| `OpenRuntime.efi` | 36,864 B | OpenCore 运行期服务 |
| `ResetNvramEntry.efi` | 57,344 B | 启动菜单中的 Reset NVRAM 入口 |
| `apfs_aligned.efi` | 718,340 B | Apple APFS 对齐驱动，用于 macOS 26 的 APFS 兼容 |

## EFI/OC/Kexts（28 个）

| kext 目录 | 版本 | Bundle ID | 大小 | 状态 | 作用 |
| --- | --- | --- | ---: | --- | --- |
| `AMFIPass.kext` | 1.4.1 | `com.dhinakg.AMFIPass` | 837,513 B | 启用 | AMFI 校验放行 |
| `AirportItlwm.kext` | 2.4.0 | `com.zxystd.AirportItlwm` | 15,925,041 B | 启用 | Intel Wi-Fi 的 Airport 接口驱动 |
| `AppleALC.kext` | 1.9.8 | `as.vit9696.AppleALC` | 4,250,299 B | 启用 | 声卡布局注入 |
| `AppleMCEReporterDisabler.kext` | 1.2 | `org.xlnc.disabler.MCEReporter` | 1,937 B | 启用 | 关闭 AppleMCEReporter |
| `BlueToolFixup.kext` | 2.7.3 | `as.acidanthera.BlueToolFixup` | 43,562 B | 启用 | 蓝牙协议栈补丁 |
| `BrightnessKeys.kext` | 1.0.4 | `as.acidanthera.BrightnessKeys` | 39,288 B | 启用 | 亮度快捷键 |
| `CpuTopologyRebuild.kext` | 2.0.2 | `org.b00t0x.CpuTopologyRebuild` | 35,541 B | 启用 | P/E 核心拓扑重建 |
| `IntelBTPatcher.kext` | 2.5.2 | `com.zxystd.IntelBTPatcher` | 43,585 B | 启用 | Intel 蓝牙固件补丁 |
| `IntelBluetoothFirmware.kext` | 2.5.0 | `com.zxystd.IntelBluetoothFirmware` | 11,508,846 B | 启用 | Intel 蓝牙固件加载 |
| `Lilu.kext` | 1.7.3 | `as.vit9696.Lilu` | 546,583 B | 启用 | 内核扩展加载框架 |
| `NVMeFix.kext` | 1.1.4 | `org.acidanthera.NVMeFix` | 53,682 B | 启用 | NVMe 电源管理修正 |
| `RTL812xLucy.kext` | 1.1.100 | `com.insanelymac.RTL812xLucy` | 419,136 B | 启用 | Realtek 2.5GbE 有线网卡 |
| `RestrictEvents.kext` | 1.1.7 | `as.vit9696.RestrictEvents` | 58,671 B | 启用 | CPU 名称与机型信息 |
| `SMCBatteryManager.kext` | 1.3.9 | `ru.usrsse2.SMCBatteryManager` | 230,800 B | 启用 | 电池状态 |
| `SMCLightSensor.kext` | 1.3.9 | `ru.usrsse2.SMCLightSensor` | 36,899 B | 启用 | 环境光传感器 |
| `SMCProcessor.kext` | 1.3.9 | `as.vit9696.SMCProcessor` | 47,802 B | 启用 | 处理器温度与频率传感器 |
| `SMCSuperIO.kext` | 1.3.9 | `ru.joedm.SMCSuperIO` | 138,708 B | 启用 | 风扇与 I/O 芯片传感器 |
| `SimpleHDA.kext` | 1.0.0 | `cn.laobamac.SimpleHDA` | 4,473,838 B | 启用 | HDA 设备简化支持 |
| `USBToolBox.kext` | 1.2.0 | `com.dhinakg.USBToolBox.kext` | 41,164 B | 启用 | USB 端口映射注入框架 |
| `UTBDefault.kext` | 1.0 | `com.dhinakg.USBToolBox.injector` | 1,306 B | 禁用 | USBToolBox 默认注入器 |
| `UTBMap-Mountainatic.kext` | 1.0.1 | `local.mountainatic.USBMap` | 5,994 B | 启用 | 本机 USB 端口映射 |
| `VirtualSMC.kext` | 1.3.9 | `as.vit9696.VirtualSMC` | 254,472 B | 启用 | SMC 模拟 |
| `VoodooI2C-7F4F.kext` | 1.0.0 | `local.mountainatic.VoodooI2C7F4F` | 1,444 B | 启用 | 本机 I2C 设备 ID 变体 |
| `VoodooI2C.kext` | 2.9.1 | `com.alexandred.VoodooI2C` | 677,639 B | 启用 | I2C 触控板总线驱动 |
| `VoodooI2CHID.kext` | 1 | `com.alexandred.VoodooI2CHID` | 248,397 B | 启用 | I2C HID 触控板驱动 |
| `VoodooPS2Controller.kext` | 2.3.8 | `as.acidanthera.voodoo.driver.PS2Controller` | 494,219 B | 部分启用（VoodooInput.kext 禁用） | PS2 键盘与触控板 |
| `WhateverGreen.kext` | 1.7.2 | `as.vit9696.WhateverGreen` | 534,820 B | 启用 | 显卡通用修正 |
| `itlwm.kext` | 2.3.0 | `com.zxystd.itlwm` | 15,988,973 B | 禁用 | Intel Wi-Fi 备用驱动 |

## EFI/OC/Resources（3 个子目录）

### Resources/Font（6 个文件，117,879 B）

```text
Font_1x.bin                                                      1,985 B
Font_1x.png                                                      3,426 B
Font_2x.bin                                                      1,988 B
Font_2x.png                                                      7,424 B
Terminus.hex                                                    51,528 B
TerminusCore.hex                                                51,528 B
```

### Resources/Image（26 个文件，3,444,358 B (3.28 MB)）

```text
Acidanthera/GoldenGate/AppleRecv.icns                           98,171 B
Acidanthera/GoldenGate/AppleTM.icns                             66,217 B
Acidanthera/GoldenGate/Background.icns                       2,552,118 B
Acidanthera/GoldenGate/BtnFocus.icns                             2,650 B
Acidanthera/GoldenGate/Cursor.icns                               1,262 B
Acidanthera/GoldenGate/Dot.icns                                    293 B
Acidanthera/GoldenGate/Enter.icns                                1,651 B
Acidanthera/GoldenGate/ExtAppleRecv.icns                        94,546 B
Acidanthera/GoldenGate/ExtAppleTM.icns                          76,078 B
Acidanthera/GoldenGate/ExtHardDrive.icns                        65,723 B
Acidanthera/GoldenGate/FirmwareSettings.icns                    98,171 B
Acidanthera/GoldenGate/HardDrive.icns                           83,952 B
Acidanthera/GoldenGate/Left.icns                                 3,048 B
Acidanthera/GoldenGate/Lock.icns                                 2,622 B
Acidanthera/GoldenGate/NetworkBoot.icns                         98,171 B
Acidanthera/GoldenGate/OpticalDrive.icns                        85,314 B
Acidanthera/GoldenGate/Password.icns                               808 B
Acidanthera/GoldenGate/Restart.icns                              5,790 B
Acidanthera/GoldenGate/Right.icns                                3,019 B
Acidanthera/GoldenGate/Selected.icns                             1,348 B
Acidanthera/GoldenGate/Selector.icns                             3,010 B
Acidanthera/GoldenGate/SetDefault.icns                           3,415 B
Acidanthera/GoldenGate/Shell.icns                               12,072 B
Acidanthera/GoldenGate/ShutDown.icns                             6,168 B
Acidanthera/GoldenGate/Tool.icns                                15,342 B
Acidanthera/GoldenGate/Windows.icns                             63,399 B
```

### Resources/Label（26 个文件，47,590 B）

```text
Apple.l2x                                                        1,733 B
Apple.lbl                                                          437 B
AppleRecv.l2x                                                    3,917 B
AppleRecv.lbl                                                      977 B
AppleTM.l2x                                                      3,149 B
AppleTM.lbl                                                        785 B
EFIBoot.l2x                                                      1,997 B
EFIBoot.lbl                                                        497 B
FirmwareSettings.l2x                                             3,989 B
FirmwareSettings.lbl                                             1,001 B
NetworkBoot.l2x                                                  2,885 B
NetworkBoot.lbl                                                    725 B
Other.l2x                                                        2,165 B
Other.lbl                                                          545 B
ResetNVRAM.l2x                                                   3,221 B
ResetNVRAM.lbl                                                     809 B
SIPDisabled.l2x                                                  4,781 B
SIPDisabled.lbl                                                  1,193 B
SIPEnabled.l2x                                                   4,661 B
SIPEnabled.lbl                                                   1,169 B
Shell.l2x                                                        2,405 B
Shell.lbl                                                          605 B
Tool.l2x                                                         1,037 B
Tool.lbl                                                           257 B
Windows.l2x                                                      2,117 B
Windows.lbl                                                        533 B
```

## docs 与仓库根文件

| 文件 | 大小 |
| --- | ---: |
| `README.md` | 22,212 B |
| `.gitignore` | 16 B |
| `docs/boot-issues.md` | 8,147 B |
| `docs/bring-up-history/Intel-映射修复测试.md` | 5,084 B |
| `docs/bring-up-history/config.Discrete-before.plist` | 39,158 B |
| `docs/bring-up-history/config.FirmwareFallback.plist` | 39,159 B |
| `docs/bring-up-history/恢复说明.txt` | 1,159 B |
| `docs/bring-up-history/最近一次接管结果.md` | 2,151 B |
| `docs/known-issues.md` | 1,085 B |

## 调试备份清理

发布前移除 54 个调试备份文件（22,757,352 B，约 21.7 MB）与 4 个 Finder 元数据文件，明细见 [boot-issues.md](boot-issues.md)。当前 EFI 目录保留的全部文件均参与实际引导。

