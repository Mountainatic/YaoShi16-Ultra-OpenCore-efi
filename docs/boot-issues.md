# 启动问题记录

本文件记录耀世 16 Ultra 在 OpenCore 适配过程中遇到的两个启动问题。前一个问题已有稳定修复，后一个问题当前配置可以完成启动，完整说明仍待在原机上补充。每个问题按「现象 → 定位线索 → 配置中的处理 → 状态」展开，便于同型号机器复现与核对。

调试期间产生的 54 个备份文件已从发布目录移出，恢复用的完整副本保留在原始 1401 U 盘。仓库中保留能够复现当前状态的最小配置与文档。

## 问题一：内核交接阶段挂起（EXITBS:START 与 Type 9 处理器）

### 现象

机器在 OpenCore 完成引导后停在一行文本上，内容为 `#【EB｜LOG:EXITBS:START】` 加时间戳。屏幕不再更新，系统无法进入 macOS。

### 定位线索

调试日志显示 OpenCore 侧的处理器识别正常：`OCCPU: Found Intel(R) Core(TM) Ultra 9 275HX`，随后是 `OCCPU: Signature C0662 Stepping 2 Model C6 Family 6 Type 0 ExtModel C ExtFamily 0 uCode 116` 与 `OCCPU: Detected Apple Processor Type: 02 -> 0905`。由此可以确定，问题不在固件识别，而在 Apple 内核接管处理器的阶段。

日志末尾给出交接状态行 `OCABC: FEXITBS 0 PRMRG 0 CSLIDE 1 MSLIDE 0 PRSRV 1 RBMAP 1 VMAP 1 APPLOS 0 RTPERMS 1 ARBAR 0 RBIO 0`，紧接着的记录为 `AAPL: #[EB|LOG:EXITBS:START] 2026-10-22T18:52:20`。日志到此为止，挂起点因此落在内核交接过程。

本机的 Core Ultra 9 275HX 属于 Arrow Lake-HX。该平台固件的 MADT 只提供 Type 9（x2APIC）处理器条目，macOS 26 的内核代码沿用较早的处理器类型判断，在交接早期即退出。配置中相关注释使用 `type-9-only firmware` 这一说法描述该固件特征，本记录中的「Type 9」即由此而来。

### 配置中的处理

当前 `config.plist` 用一组 CPUID 覆盖与 Booter 设置把处理器描述成 macOS 能够识别的形态，具体项目如下。

| 项目 | 位置 | 当前值 | 作用 |
| --- | --- | --- | --- |
| CPUID 覆盖 | `Kernel > Emulate > Cpuid1Data` | `55 06 0a 00` 加 12 个 `00`，即 leaf 1 EAX = `0x000A0655` | 让内核按 Alder Lake 读取处理器特征 |
| CPUID 掩码 | `Kernel > Emulate > Cpuid1Mask` | `ff ff ff ff` 加 12 个 `00` | 按位写入上述覆盖值 |
| 内存映射重建 | `Booter > Quirks > RebuildAppleMemoryMap` | `true` | 交接前重建 Apple 内存映射 |
| 虚拟地址映射 | `Booter > Quirks > SetupVirtualMap` | `true` | 建立内核运行期虚拟映射 |
| Slide 选择 | `Booter > Quirks > ProvideCustomSlide` | `true` | 允许自定义内核 slide |
| Slide 上限 | `Booter > ProvideMaxSlide` | `0` | 由 OpenCore 自动选择 |
| 安全模式 slide | `Booter > Quirks > EnableSafeModeSlide` | `true` | 为安全模式保留 slide 处理 |
| 运行时权限同步 | `Booter > Quirks > SyncRuntimePermissions` | `true` | 同步页表权限 |
| DMAR 表剔除 | `ACPI > Delete > DMAR` | 已启用 | 配置注释：`Drop VT-d DMAR (Core Ultra hangs at the kernel hand-off with it)` |

其中 DMAR 剔除单独列出，原因在配置注释里写得很清楚：该固件携带的 VT-d DMAR 表会让 Core Ultra 平台在内核交接处挂起。移除这张表后，VT-d 可以在 BIOS 中保持开启，Apple 侧的 IOMMU 相关逻辑也不会提前中断。

### 状态

问题一已解决。当前 `config.plist` 可以稳定进入 macOS，同样的取值在 2026-10-22 之后的多份配置中保持一致。

### 需要说明的一点

备份文件 `config_before_cpu_275hx_20261023.plist` 与当前配置一样包含上述 CPUID 覆盖，因此该文件用于回退到修复后的已知可用状态。修复前的现场由 1401 U 盘上的 `EFI-before` 完整目录保留，需要对比原始配置时以该目录为准。

## 问题二：位偏移（MADT 与 ACPI 区域修复）

### 现象

完成问题一的修复后，启动过程仍存在一个由机器所有者概括为「位偏移」的问题。该问题与固件 MADT 的处理器条目布局、ACPI 表的区域与偏移字段相关。问题二的具体日志、触发条件与原机定位过程记录较少，本文件先整理当前配置中与之对应的处理痕迹与历史备份演进，完整机理说明留待所有者补充。

### 配置中的处理

依据配置注释、参数取值与备份文件命名，当前 `config.plist` 从三个方向处理这组问题。

第一处是 MADT 替换。`ACPI > Add` 中的 `APIC-LAPIC.aml`（658 B）注释为 `24 actual CPU UID/APIC mappings; enabled x2APIC entries represented as Local APIC`；`ACPI > Delete` 中的 `APIC` 条目注释为 `Replace firmware MADT with same-machine Local APIC compatible table`。这组设置用同机采集的 Local APIC 表替换固件 MADT，并把其中启用的 x2APIC 条目以 Local APIC 形式呈现。

第二处是 ACPI 区域偏移重算。`ACPI > Quirks > RebaseRegions` 取值为 `true`，OpenCore 在加载 ACPI 表时重新计算表内区域字段的偏移。与之配合的还有 `DSDT.aml` 的加载方式：该表注释为 `firmware DSDT with forward Scopes deferred (loads in macOS's ACPICA)`，指向 macOS 的 ACPICA 解释器推迟处理前向 Scope 声明。

第三处是补丁与替换。`ACPI > Patch` 中的 `Remove conditional ACPI scope declaration` 将查找值 `a0 00 00 00 92 93 50 43 48 41 00` 替换为 11 个 `0xA3`（NOP），启用计数为 1，用于移除固件 DSDT 中会造成前向 Scope 的条件声明。`Kernel > Patch` 中另有一条名为 `Tahoe 25.6: count enabled x2APIC MADT entries on this type-9-only firmware` 的内核补丁，Base 为 `_acpi_count_enabled_logical_processors`，作用范围 25.6.0 起。该补丁在当前配置中以 `Enabled = false` 保留，作为历史处理记录。

### 历史备份的演进

调试期间保存的配置与 ACPI 备份按时间顺序展示了处理过程。

| 文件 | 时间 | 主要特征 |
| --- | --- | --- |
| `config_x2apic_count_backup.plist` | 2026-10-22 21:26 | 10 张 ACPI 表，无 `APIC-LAPIC.aml`；`ACPI > Delete` 只含 DMAR；无 x2APIC 内核补丁 |
| `config_madt_compat_backup.plist` | 2026-10-23 12:05 | 10 张 ACPI 表；x2APIC 计数内核补丁处于启用状态 |
| `config_acpi_region_fix_backup.plist` | 2026-10-23 12:42 | 加入 `APIC-LAPIC.aml` 并删除固件 APIC 表；x2APIC 内核补丁转为禁用；`RebaseRegions = false` |
| `DSDT_region_fix_backup.aml` | 2026-10-22 16:07 | 与当前 `DSDT.aml` 同为 596,487 B；两者相差 13 字节，其中第 10 字节为表校验和 `0xF8 → 0x4C`，其余 12 处为 `_OSI` 改名 `XOSI` 的字节替换（`0x5F → 0x58`） |
| `config.plist`（当前） | 2026-10-24 23:43 | 11 张 ACPI 表；`RebaseRegions = true`；x2APIC 内核补丁保留且禁用 |

由此可以看到一条清晰的演进路线：先在固件 MADT 上做内核计数补丁，随后改为用同机采集的 Local APIC 表整体替换 MADT，最后打开 ACPI 区域重算并稳定在当前配置。

### 状态

当前配置可以完成启动并进入桌面。问题二的完整描述与最终验证方法仍待补充，下面留一条 TODO 供所有者确认。

> TODO(owner)：请补充「位偏移」的准确描述，包括现象、日志、定位过程、最终修复涉及的具体偏移位置与验证结果。补充完成后，本节替换为完整说明。

## 调试备份的清理记录

发布前从 `EFI/` 目录移除了 54 个调试备份文件与 4 个 Finder 元数据文件，并删除空的 `EFI/OC/Tools/` 目录。移除总量为 22,757,352 B，约 21.7 MB。

| 类别 | 数量 | 说明 |
| --- | --- | --- |
| `OpenCore_*_backup.efi` | 39 | 其中 26 个为真实文件（25 个 831,488 B、1 个 872,448 B），13 个为零字节占位；含早期 `probe*.efi` 探测构建 |
| `OpenRuntime_backup.efi` | 1 | 36,864 B |
| `DSDT_region_fix_backup.aml` | 1 | 596,487 B |
| `config_*backup*.plist` | 13 | 合计 464,353 B |
| Finder 元数据 | 4 | `.contentFlavour` 与 `.contentVisibility`，各 8 B |

恢复用的完整副本保留在原始 1401 U 盘，路径包括 `EFI-before`、`XeGfx-Recovery-20261023` 与调试日志目录。
