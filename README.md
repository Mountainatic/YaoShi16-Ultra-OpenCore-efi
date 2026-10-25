# 耀世 16 Ultra OpenCore EFI

机械革命耀世 16 Ultra（Yaoshi 16 Ultra）在 macOS Tahoe 26 下的 OpenCore 配置仓库。本仓库提供一套可启动的 EFI 目录、脱敏后的 `config.plist`、组件清单与适配记录，给同型号机器的黑苹果安装提供可复现的起点。仓库地址为 [Mountainatic/YaoShi16-Ultra-OpenCore-efi](https://github.com/Mountainatic/YaoShi16-Ultra-OpenCore-efi)。

本仓库的 EFI 目录可以直接复制到 FAT32 格式的 ESP 分区。`config.plist` 中的 SMBIOS 字段已经换成占位符，使用前请按后文步骤生成自己的序列号。NVIDIA 显示驱动与 Intel iGPU 驱动是独立项目，按照后文说明单独获取。

## 目录

- [机器配置](#机器配置)
- [当前状态](#当前状态)
- [进行中的工作：Intel Xe iGPU](#进行中的工作intel-xe-igpu)
- [EFI 内容总览](#efi-内容总览)
- [使用步骤](#使用步骤)
- [已知问题](#已知问题)
- [与默认 OpenCore 示例的差异](#与默认-opencore-示例的差异)
- [仓库结构](#仓库结构)
- [组件、版本与上游项目](#组件版本与上游项目)
- [许可与免责](#许可与免责)

## 机器配置

| 项目 | 规格 |
| --- | --- |
| 机型 | 机械革命耀世 16 Ultra（MECHREVO Yaoshi 16 Ultra） |
| 处理器 | Intel Core Ultra 9 275HX（Arrow Lake-HX，24 核 32 线程级别，P+E 混合架构） |
| 核芯显卡 | Intel Xe-LPG，设备 ID `8086:7D67` |
| 独立显卡 | NVIDIA GeForce RTX 5080 Laptop 16 GB |
| 内存 | 64 GB |
| 硬盘 | 1 TB NVMe SSD |
| 屏幕 | 16 英寸 2560x1600 240 Hz，面板 BOE NE160QDM-NZL |
| 无线网卡 | Intel Wi-Fi + 蓝牙（CNVi / M.2 组合） |
| 有线网卡 | Realtek 2.5GbE，驱动为 RTL812xLucy（具体型号以本机为准） |
| 操作系统 | macOS Tahoe 26.7.1（25G241） |
| 引导器 | OpenCore 1.0.8 |

日常使用模式为 1920x1200 HiDPI。内屏当前由独立显卡路径提供画面；Intel iGPU 的驱动开发仍在进行，其 bring-up 阶段使用 2560x1600 60 Hz 级别的时序。

## 当前状态

| 功能 | 状态 | 实现组件 | 备注 |
| --- | --- | --- | --- |
| 启动 macOS Tahoe | 可用 | OpenCore 1.0.8，CPUID 覆盖 | 见「已知问题」问题一 |
| CPU 频率与电源管理 | 可用 | SSDT-PLUG-ALT、SMCProcessor、SMCSuperIO | XCPM 生效 |
| P/E 核心拓扑 | 可用 | CpuTopologyRebuild 2.0.2 | P 核与 E 核分别映射，`hw.logicalcpu` 为 24 |
| CPU 型号显示 | 可用 | RestrictEvents | `revpatch=sbvmm,cpuname`、`revcpu=1` |
| 核显 | 开发中 | 独立项目 | 见下一节 |
| 独显显示与 Metal | 可用 | NullMoth NvGfx 2.5.3（装在 macOS 内） | 内屏经该路径输出，HiDPI 由 `nvfbhidpi=1` 打开 |
| 内屏 HiDPI | 可用 | NvGfx 显示路径 | 日常 1920x1200 |
| Wi-Fi | 可用 | AirportItlwm 2.4.0 | `itlwm_he=1`；`itlwm.kext` 作为备用方案保留并禁用 |
| 蓝牙 | 可用 | IntelBluetoothFirmware、IntelBTPatcher、BlueToolFixup | 组合加载 |
| 电池 | 可用 | SMCBatteryManager | 电量与充电状态 |
| 触控板 | 可用 | VoodooI2C 2.9.1、VoodooI2C-7F4F、VoodooI2CHID | 含 VoodooGPIO 插件 |
| 键盘与亮度键 | 可用 | VoodooPS2Controller 2.3.8、BrightnessKeys | PS2 与 I2C 两条输入路径并存 |
| USB 端口 | 可用 | USBToolBox、UTBMap-Mountainatic | 端口映射按本机生成 |
| NVMe 电源管理 | 可用 | NVMeFix 1.1.4 | |
| AMFI 放宽 | 可用 | AMFIPass 1.4.1、boot-args | 允许加载未签名驱动 |
| 有线网卡 | 驱动已加载，待验证 | RTL812xLucy 1.1.100 | 与 RTL8125 系列对应 |
| 环境光传感器 | 驱动已加载，待验证 | SMCLightSensor 1.3.9 | |
| 睡眠与唤醒 | 未测试 | — | 尚未研究，暂无结论 |
| 核显内屏输出 | 开发中 | 独立项目 | 当前 60 Hz 级别时序 |

## 进行中的工作：Intel Xe iGPU

Intel Xe-LPG 核显的 macOS 驱动是一个独立项目，仓库地址为 [Mountainatic/intel-xe-lpg-igpu-macos](https://github.com/Mountainatic/intel-xe-lpg-igpu-macos)。该项目的目标是为 `8086:7D67` 提供可用的显示与加速路径，当前处于 bring-up 阶段，内屏已能使用 2560x1600 60 Hz 级别的时序。

内屏画面目前由独立显卡路径承担，核显项目在此基础上逐步接管。接管过程的测试记录保存在本仓库的 `docs/bring-up-history/` 目录：

- `Intel-映射修复测试.md`：XR 映射修复的测试记录与回滚路径；
- `最近一次接管结果.md`：level 4 插件接管后的画面残影现象、窗口服务器状态与显示器修复过程；
- `恢复说明.txt`：接管失败时回到独立显卡路径的操作步骤。

这两份记录的细节以原机实测为准，供同型号机器排障时参考。

## EFI 内容总览

`EFI/` 目录共 150 个文件，约 60 MB。其中 `EFI/OC/Kexts/` 占 54 MB，`EFI/OC/Resources/` 占 3.6 MB，其余为引导文件与配置。

### 引导文件

| 文件 | 大小 | 说明 |
| --- | --- | --- |
| `EFI/BOOT/BOOTx64.efi` | 52 KB | OpenCore 的 UEFI 启动入口 |
| `EFI/OC/OpenCore.efi` | 831,488 B | OpenCore 1.0.8 主体 |

### ACPI 表（11 张）

| 文件 | 大小 | 作用 |
| --- | --- | --- |
| `DSDT.aml` | 596,487 B | 固件 DSDT；已按本次固件的实际 GNVS 运行时地址修正 `SystemMemory` 区域，避免 AML 跨页访问错误的平台状态 |
| `APIC-LAPIC.aml` | 658 B | Type 9 / x2APIC 兼容 MADT：将 24 个有效 Processor Local x2APIC 条目转换为传统 Local APIC 兼容表示，同时保留 UID、真实 APIC ID 与 enabled 状态 |
| `SSDT-PLUG-ALT.aml` | 3,112 B | 处理器电源管理注入（替代方案） |
| `SSDT-EC.aml` | 125 B | 嵌入式控制器描述 |
| `SSDT-GPI0.aml` | 124 B | I2C GPIO 设备描述 |
| `SSDT-MCHC.aml` | 105 B | 内存控制器中枢描述 |
| `SSDT-RTCAWAC.aml` | 78 B | RTC 与 AWAC 时钟设备 |
| `SSDT-SBUS.aml` | 128 B | SMBus 设备描述 |
| `SSDT-USB-Reset.aml` | 181 B | USB 复位属性 |
| `SSDT-USBX.aml` | 166 B | USB 供电属性 |
| `SSDT-XOSI.aml` | 219 B | `_OSI` 到 `XOSI` 的改名配合表 |

### UEFI 驱动（4 个）

| 文件 | 大小 | 作用 |
| --- | --- | --- |
| `HfsPlus.efi` | 37,892 B | Apple HFS+ 文件系统驱动 |
| `OpenRuntime.efi` | 36,864 B | OpenCore 运行期服务 |
| `ResetNvramEntry.efi` | 57,344 B | 启动菜单中的 Reset NVRAM 入口 |
| `apfs_aligned.efi` | 718,340 B | Apple APFS 对齐驱动，用于 macOS 26 的 APFS 兼容 |

`UEFI > APFS > EnableJumpstart` 取值为 `false`，与 `apfs_aligned.efi` 配合使用。

### 内核扩展（28 个）

`EFI/OC/Kexts/` 下为 28 个顶层 kext，其中 26 个启用，2 个按设计禁用：

- `itlwm.kext`：备用无线方案，与 AirportItlwm 二选一；
- `UTBDefault.kext`：USBToolBox 的默认注入器，由本机端口映射 UTBMap-Mountainatic 替代。

`Kernel > Add` 中另有 1 个插件保持禁用：`VoodooPS2Controller.kext/Contents/PlugIns/VoodooInput.kext`，输入插件由 VoodooI2C 的同名插件统一提供。`Kernel > Add` 共 35 个条目，覆盖 28 个 kext 及其插件。逐文件清单与版本见 [docs/efi-inventory.md](docs/efi-inventory.md)。

### 引导参数

`NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > boot-args` 原样保留为：

```
debug=0x100 keepsyms=1 -vi2c-force-polling -ibtcompatbeta revpatch=sbvmm,cpuname nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0 gfxrst=4 nvfbhidpi=1 nvrmacpibl=1 itlwm_he=1 revcpu=1
```

其中 `debug=0x100 keepsyms=1` 打开调试输出与符号保留，`revpatch=sbvmm,cpuname` 与 `revcpu=1` 驱动 RestrictEvents 的机型与 CPU 名称显示，`amfi=0x80` 与 `amfi_get_out_of_my_way=0x1` 放宽 AMFI 校验，`itlwm_he=1` 对应 AirportItlwm 的 HE 支持，`nvfb`、`nvaccel`、`nvfbheads`、`nvfbhidpi`、`nvrmacpibl`、`gfxrst` 与 `-nvkmsnosmooth` 属于 NullMoth NvGfx 的显示参数。显卡参数的完整含义以 [NullMoth nvidia-macos-driver](https://github.com/nullmoth/nvidia-macos-driver) 的文档为准。

### 其他关键设置

| 位置 | 取值 | 说明 |
| --- | --- | --- |
| `PlatformInfo > Generic` | 占位符 | Serial、MLB、UUID、ROM 均待生成 |
| `PlatformInfo > Generic > SystemProductName` | `MacBookPro16,2` | 机型标识 |
| `PlatformInfo > Generic > ProcessorType` | `0x0F01` | 自定义处理器类型 |
| `Kernel > Emulate > Cpuid1Data` | `55060a00` 加 12 个 `00` | CPUID leaf 1 覆盖 |
| `Kernel > Emulate > Cpuid1Mask` | `ffffffff` 加 12 个 `00` | 覆盖掩码 |
| `ACPI > Quirks > RebaseRegions` | `true` | ACPI 区域偏移重算 |
| `ACPI > Delete` | `DMAR`、`APIC` | 移除固件 DMAR 与 MADT |
| `Misc > Security > SecureBootModel` | `Disabled` | 关闭安全启动模型 |
| `Misc > Security > ScanPolicy` | `0` | 允许扫描所有启动项 |
| `Misc > Boot > PickerMode` | `Builtin` | 文本选择菜单 |
| `Misc > Boot > Timeout` | `5` | 菜单停留 5 秒 |

## 使用步骤

1. 准备一个 16 GB 以上的 U 盘，分区表使用 GPT，EFI 分区格式为 FAT32，容量 200 MB 以上。
2. 将本仓库的 `EFI` 目录整体复制到 U 盘 EFI 分区根目录。复制完成后，U 盘上出现 `EFI/BOOT/BOOTx64.efi` 与 `EFI/OC/`。
3. 进入 BIOS 完成基础设置：
   - Secure Boot 设为关闭或 Setup 模式；
   - 启动模式选择 UEFI，CSM 关闭；
   - Above 4G Decoding 开启；
   - Resizable BAR 保持关闭或自动，与 `ResizeAppleGpuBars=0`、`ResizeGpuBars=-1` 对应；
   - VT-d 可以保持开启，固件 DMAR 表已由 ACPI 删除规则移除；
   - CFG Lock 有选项时关闭，没有选项时依靠 `AppleXcpmCfgLock=true`。
4. 生成自己的 SMBIOS。使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 选择 `MacBookPro16,2`，把生成的 Serial、MLB、SystemUUID、ROM 填入 `PlatformInfo > Generic`。保留 `SystemProductName` 与 `ProcessorType` 两项原值。编辑 plist 推荐使用 [ProperTree](https://github.com/corpnewt/ProperTree)。
5. 保留 `boot-args` 原文。显示相关参数与 NvGfx 驱动配套，无线参数与 AirportItlwm 配套。
6. 从 U 盘启动，在 OpenCore 菜单选择 macOS。首次启动失败时，可以在菜单中进入 Reset NVRAM 后重试。
7. 安装 NVIDIA 显示驱动。驱动由 NullMoth 项目的 [2.5.3 版本](https://github.com/nullmoth/nvidia-macos-driver/releases/tag/v2.5.3) 提供，安装到 macOS 系统卷并重建缓存。驱动接管显示后，把 `Kernel > Block` 中 `com.apple.iokit.IONDRVSupport` 的 `Enabled` 改为 `true`。该条目当前为 `false`，注释为 `NullMoth display takeover after driver installation (Tahoe)`。
8. 显卡模式按 BIOS 能力选择。本机 BIOS 提供 Hybrid 与 Discrete 两种模式，两种模式下的启动都需要单独验证。切换模式之前请备份当前 EFI，恢复流程见 `docs/bring-up-history/恢复说明.txt`。
9. macOS 安装器请通过 Apple 官方渠道获取，本仓库只包含 EFI 配置。

## 已知问题

### 问题一：Type 9 / x2APIC 兼容

原始 MADT 中，24 个有效 CPU 全部使用 Type 9 `Processor Local x2APIC` 条目，Type 0 `Processor Local APIC` 条目数量为 0。XNU 的早期处理器枚举路径只识别传统 Type 0 表示，因此首先得到：

```text
max_cpus_from_firmware = 0
```

继续推进启动后，又暴露出后续拓扑注册与 MADT 枚举结果不一致：

```text
x86_validate_topology() 24 threads but 1 registered from MADT
```

最终修复没有停留在早期 CPU 计数补丁，而是将固件中有效的 Type 9 CPU 转换为传统 Local APIC 兼容表示，并保留每个处理器的 UID、真实 APIC ID 与 enabled 状态。这样，XNU 的早期 CPU 计数和后续 AppleACPICPU 注册读取的是同一套兼容 MADT。

修正后，24 核拓扑可以完整注册，XNU 真正继续进入 Recovery。当前配置通过删除固件 `APIC` 表并加载 `APIC-LAPIC.aml` 提供这套兼容表示。

### 问题二：DSDT / GNVS 运行时地址偏移

此前替换 DSDT 中的 GNVS `SystemMemory` 地址时，写入值比本次固件实际运行时地址偏移了 `0x1000`。这一页级偏移会使 ACPI AML 从错误的物理页读取或写入平台状态，因此可能同时影响 PCI、USB、I2C 与电源相关资源。

将 GNVS 地址修正为本次固件的实际运行时地址后，原机观察到：

- USB 鼠标恢复；
- 内置 NVMe 正常枚举；
- Recovery 可以正常操作。

因此当前 DSDT 使用已经校正的 GNVS 运行时地址，相关 ACPI 区域与后续设备枚举均以该地址为基础。

### 其他注意事项

- Intel Xe 核显的驱动仍在开发，内屏当前走独立显卡路径，核显 bring-up 使用 60 Hz 级别时序。
- 显卡模式切换（Hybrid 与 Discrete）后需要重新验证启动与显示输出。
- 有线网卡与环境光传感器的驱动已经加载，功能验证尚未逐项记录。
- 睡眠与唤醒尚未研究，当前状态记为未测试。
- NVIDIA 驱动安装在 macOS 内，重装系统或重建内核缓存后需要重新安装，并同步检查 `Kernel > Block` 条目。

## 与默认 OpenCore 示例的差异

与 OpenCore 官方 `Docs/Sample.plist` 相比，本配置围绕本机硬件做了以下调整：

1. 增加 CPUID 覆盖。Arrow Lake-HX 不在 Apple 支持列表中，`Cpuid1Data` 与 `Cpuid1Mask` 提供处理器特征覆盖，配合 `Booter > Quirks` 的完整内存映射设置。
2. 自备 ACPI 表。仓库带 1 张修正后的 DSDT 与 10 张 SSDT；其中 `APIC-LAPIC.aml` 将原始 MADT 中 24 个有效 Type 9 x2APIC CPU 转换为传统 Local APIC 兼容表示，并保留 UID、真实 APIC ID 与 enabled 状态。
3. 修正 DSDT 的 GNVS 运行时地址。原替换值相对本次固件实际地址偏移 `0x1000`；校正后 USB 鼠标、内置 NVMe 与 Recovery 操作恢复正常。
4. 增加 ACPI 删除与替换。固件 `DMAR` 与原始 `APIC` 表被移除，其中 `APIC` 由兼容 MADT 接管；`RebaseRegions` 保持开启，用于配合 ACPI 区域重定位。
5. 增加 `ResizeAppleGpuBars=0` 与 `ResizeGpuBars=-1`。两者与 NVIDIA 显卡的 BAR 配置配合。
6. 使用 `MacBookPro16,2` 机型与 `ProcessorType=0x0F01`。机型用于电源管理与显示服务，处理器类型用于 CPU 名称显示。
7. 28 个 kext 的完整驱动集合。默认示例只带 Lilu、VirtualSMC、WhateverGreen，本配置补齐无线、蓝牙、触控板、电池、USB 映射、NVMe、AMFI 与 CPU 拓扑等组件。
8. 替换 UEFI 驱动。`HfsPlus.efi` 使用 Apple 的 HFS+ 驱动，`apfs_aligned.efi` 用于 macOS 26 的 APFS 兼容，`ResetNvramEntry.efi` 提供菜单入口。
9. 增加大段 `boot-args`。显示、无线、AMFI 与调试参数按本机组合保留。
10. 保留 OpenCanopy 资源但使用文本菜单。`Resources/` 中保留 Font、Image、Label，`PickerMode` 取 `Builtin`。
11. 保留两条禁用条目。`Kernel > Patch` 的 x2APIC 计数补丁与 `Kernel > Block` 的 `IONDRVSupport` 排除条目以 `Enabled=false` 保留，作为历史处理记录与驱动接管开关。

## 仓库结构

```text
YaoShi16-Ultra-OpenCore-efi/
├── README.md
├── .gitignore
├── EFI/                         约 60 MB
│   ├── BOOT/
│   │   └── BOOTx64.efi
│   └── OC/
│       ├── OpenCore.efi
│       ├── config.plist         脱敏模板
│       ├── ACPI/                11 张表
│       ├── Drivers/             4 个 UEFI 驱动
│       ├── Kexts/               28 个 kext
│       └── Resources/           Font / Image / Label
└── docs/
    ├── boot-issues.md           两个启动问题的记录
    ├── known-issues.md          音频与睡眠等非启动问题
    ├── efi-inventory.md         逐文件清单与版本
    └── bring-up-history/        5 份原机记录
```

## 组件、版本与上游项目

版本读取自各 kext 与驱动的 `Info.plist`。逐文件大小见 [docs/efi-inventory.md](docs/efi-inventory.md)。

### 核心框架与系统服务

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| Lilu | 1.7.3 | 内核扩展加载框架 | [acidanthera/Lilu](https://github.com/acidanthera/Lilu) |
| VirtualSMC | 1.3.9 | SMC 模拟 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) |
| SMCProcessor | 1.3.9 | 处理器温度与频率传感器 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) |
| SMCSuperIO | 1.3.9 | 风扇与 I/O 芯片传感器 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) |
| SMCBatteryManager | 1.3.9 | 电池状态 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) |
| SMCLightSensor | 1.3.9 | 环境光传感器 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) |
| AMFIPass | 1.4.1 | AMFI 校验放行 | [dhinakg/AMFIPass](https://github.com/dhinakg/AMFIPass) |
| AppleMCEReporterDisabler | 1.2 | 关闭 AppleMCEReporter | XLNC 发布，社区分发 |

### 处理器与机型

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| CpuTopologyRebuild | 2.0.2 | P/E 核心拓扑重建 | [b00t0x/CpuTopologyRebuild](https://github.com/b00t0x/CpuTopologyRebuild) |
| RestrictEvents | 1.1.7 | CPU 名称与机型信息 | [acidanthera/RestrictEvents](https://github.com/acidanthera/RestrictEvents) |

### 图形与音频

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| WhateverGreen | 1.7.2 | 显卡通用修正 | [acidanthera/WhateverGreen](https://github.com/acidanthera/WhateverGreen) |
| AppleALC | 1.9.8 | 声卡布局注入 | [acidanthera/AppleALC](https://github.com/acidanthera/AppleALC) |
| SimpleHDA | 1.0.0 | HDA 设备简化支持 | [laobamac/SimpleHDA](https://github.com/laobamac/SimpleHDA) |

### 无线与网络

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| AirportItlwm | 2.4.0 | Intel Wi-Fi 的 Airport 接口驱动 | [OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm) |
| itlwm | 2.3.0 | Intel Wi-Fi 备用方案，已禁用 | [OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm) |
| IntelBluetoothFirmware | 2.5.0 | Intel 蓝牙固件加载 | [OpenIntelWireless/IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) |
| IntelBTPatcher | 2.5.2 | Intel 蓝牙固件补丁 | [OpenIntelWireless/IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) |
| BlueToolFixup | 2.7.3 | 蓝牙协议栈补丁 | [acidanthera/BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) |
| RTL812xLucy | 1.1.100 | Realtek 2.5GbE 有线网卡 | [Mieze/RTL812xLucy](https://github.com/Mieze/RTL812xLucy) |

### 输入设备

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| VoodooI2C | 2.9.1 | I2C 触控板总线驱动 | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooI2C-7F4F | 1.0.0 | 本机 I2C 设备 ID 变体 | 基于 VoodooI2C 本地修改 |
| VoodooI2CHID | 1 | I2C HID 触控板驱动 | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooGPIO | 1.1 | GPIO 控制器插件 | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooI2CServices | 1 | I2C 服务插件 | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooInput | 1.1.7 | 输入事件中转插件 | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooPS2Controller | 2.3.8 | PS2 键盘与触控板 | [acidanthera/VoodooPS2](https://github.com/acidanthera/VoodooPS2) |
| BrightnessKeys | 1.0.4 | 亮度快捷键 | [acidanthera/BrightnessKeys](https://github.com/acidanthera/BrightnessKeys) |

### 存储与 USB

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| NVMeFix | 1.1.4 | NVMe 电源管理修正 | [acidanthera/NVMeFix](https://github.com/acidanthera/NVMeFix) |
| USBToolBox | 1.2.0 | USB 端口映射注入框架 | [USBToolBox/kext](https://github.com/USBToolBox/kext) |
| UTBMap-Mountainatic | 1.0.1 | 本机 USB 端口映射 | 使用 [USBToolBox](https://github.com/USBToolBox/tool) 在本机生成 |
| UTBDefault | 1.0 | 默认注入器，已禁用 | [USBToolBox/kext](https://github.com/USBToolBox/kext) |

### OpenCore 与 UEFI 驱动

| 组件 | 版本 | 作用 | 上游项目 |
| --- | --- | --- | --- |
| OpenCorePkg | 1.0.8 | `OpenCore.efi`、`BOOTx64.efi`、`OpenRuntime.efi`、`ResetNvramEntry.efi` | [acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) |
| HfsPlus.efi | Apple 原始二进制 | HFS+ 文件系统驱动 | 随 [acidanthera/OcBinaryData](https://github.com/acidanthera/OcBinaryData) 分发 |
| apfs_aligned.efi | Apple 原始二进制 | macOS 26 的 APFS 兼容驱动，取自 macOS Sequoia 的 `apfs_align.efi` | 讨论见 [acidanthera/bugtracker#2499](https://github.com/acidanthera/bugtracker/issues/2499) |

### 配套项目

| 项目 | 版本 | 说明 |
| --- | --- | --- |
| [NullMoth nvidia-macos-driver](https://github.com/nullmoth/nvidia-macos-driver) | 2.5.3 | NVIDIA 显示驱动，安装在 macOS 内，本仓库不包含其文件 |
| [Mountainatic/intel-xe-lpg-igpu-macos](https://github.com/Mountainatic/intel-xe-lpg-igpu-macos) | 开发中 | Intel Xe-LPG 核显驱动项目 |

## 许可与免责

本仓库内的原创文档（`README.md`、`docs/`）由 Mountainatic 维护，以 MIT 许可发布，全文见 `LICENSE-MIT`。`EFI/` 内的第三方组件遵循各自上游许可：OpenCore 为 BSD-3-Clause，Apple UEFI 二进制版权归 Apple，各 kext 见其上游仓库。完整说明见 `NOTICE.md`。

NullMoth NvGfx 驱动采用 PolyForm Noncommercial License 1.0.0，仅限非商业用途，其文件安装到 macOS 系统卷，本仓库只保留引导参数。`HfsPlus.efi` 与 `apfs_aligned.efi` 为 Apple 提供的 UEFI 驱动二进制，版权归 Apple 所有，随社区 EFI 惯例分发；需要更严格的合规处理时，可以在本地替换。

`config.plist` 中的 Serial、MLB、SystemUUID 与 ROM 已替换为占位符。每台机器应使用 GenSMBIOS 生成独立的一套值，避免多台机器共用同一 SMBIOS。

修改 EFI 前请备份 ESP 分区。切换显卡模式、更新 BIOS 与重建内核缓存存在风险，操作前请确认机器的保修状态与恢复手段。
