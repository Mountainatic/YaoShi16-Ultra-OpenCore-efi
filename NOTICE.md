# 第三方组件与许可说明

本仓库的 `README.md` 与 `docs/` 文本由 Mountainatic 原创，以 MIT 许可发布（全文见 `LICENSE-MIT`）。`EFI/` 目录内是引导所需的第三方二进制与配置，它们遵循各自上游项目的许可，本仓库只做整理与脱敏。

## 引导主体

- OpenCore 1.0.8：Acidanthera 的 OpenCorePkg，BSD-3-Clause。
- `EFI/BOOT/BOOTx64.efi`、`EFI/OC/OpenCore.efi`、`EFI/OC/Drivers/` 下的 OpenRuntime.efi 等：随 OpenCore 发布，遵循 OpenCorePkg 的许可。

## Apple 提供的 UEFI 二进制

- `EFI/OC/Drivers/HfsPlus.efi`、`EFI/OC/Drivers/apfs_aligned.efi`：Apple 提供的 UEFI 驱动二进制，版权归 Apple 所有；本仓库按社区 EFI 惯例随配置分发，需要更严格的合规处理时可在本地替换。

## kext 与驱动

`EFI/OC/Kexts/` 内各 kext 均由各自上游项目发布，遵循其原始许可。README 的组件清单列出了每个 kext 的上游仓库，使用与再分发前请阅读对应仓库的许可条款。NullMoth NvGfx 显卡驱动（XeGfx）采用 PolyForm Noncommercial License 1.0.0，仅限非商业用途；本仓库不含其二进制，只在 `config.plist` 的引导参数与 `Kernel > Block` 说明里引用。

## 配置整理

`config.plist` 基于 OpenCore 官方示例配置整理，经本机适配与脱敏（Serial、MLB、SystemUUID、ROM 已替换为占位符）。示例配置本身遵循 OpenCorePkg 的 BSD-3-Clause。
