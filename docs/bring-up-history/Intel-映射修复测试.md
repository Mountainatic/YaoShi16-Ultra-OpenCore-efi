# Intel 映射与 XR 修复测试包

本轮针对 NVIDIA 对照发现的缺口修复；保持手动、headless 默认启动。已安装到磁盘，状态 installed_pending_manual_reboot。已安装插件的真实 GPU 回读也通过。新内核尚未运行，默认桌面接管继续关闭。

## 修改

1. 接管流程从 level 3 修正为 level 4，并强制检查 debug.xegfx_aperture_enabled=1 后才可能创建 WindowServer 准入。DESKTOP-HANDOFF-FAILED.json 仍保留，旧的桌面接管流程继续拒绝运行，强制注销脚本仍停用。
2. Intel 的普通资源只返回原 IOBufferMemoryDescriptor，校验长度与可访问的系统内存；只有固件 scanout 可建立真实物理窗口描述符。拒绝将 0x400000000000 起的资源 cookie 当作 scanout 物理地址，拒绝长度溢出及过小 backing。关闭开关时，即使缓存了描述符也拒绝返回。
3. 新增独立 debug.xegfx_aperture 开关和只读 enabled/system/scanout/refused/closed 计数。只有没有显示头、没有显示订阅、没有 armed/scanout 的 headless 状态能单独开关。
4. 新增 debug.xegfx_aperture_probe=1：分配 1 MiB 系统内存，通过同一个描述符工厂建立 CPU 映射，双向写入并逐字检查；同时验证 cookie 冒充物理地址和过小 backing 均被拒绝。它不构造私有 family 资源、不提交 GPU 命令、不映射 BAR、不激活显示管线。
5. 插件补齐四种 XR 纹理格式，后端表示与已部署 NVIDIA 2.5.3 对齐。BGRA10_XR 用半浮点后端，BGR10_XR 用 10:10:10:2；这是渲染兼容表示，并非 Apple 原始 XR 字节打包/完整色彩语义的实现。

## 已完成的测试

- macOS 26 分支加速器使用 SDK 15.4 编译；862 个未定义符号均由声明的公开库提供。
- 与已安装 NVAccel 的公开定义符号交集只有标准 __start、__stop 和 _kmod_info；无驱动类/函数冲突。
- 修正版与独立测试版插件均构建并签名。测试版使用独立 XeTrialMTL 类名，在测试进程手动创建目标 PCI 8086:7d67 设备，未替换系统注册的插件。
- 新插件的 IOSurface 512×512、572×572、2560×1600 GPU 清屏与全像素 CPU 回读通过。
- BGRA10_XR、BGRA10_XR_sRGB、BGR10_XR、BGR10_XR_sRGB 全部成功创建纹理及渲染管线，实际着色器绘制绿色三角形覆盖画面后回读，错误像素均为 0。
- 正式 XeMTL 类名版本的 1024 值计算和三角形绘制通过，错误 0、中心绿色、角落红色、512 个绿色像素。
- 这些结果验证了当前后端表示下的 GPU 执行；不能证明 Apple XR 原始字节语义、HDR 或跨进程零复制正确，也不能替代桌面稳定性验证。

所有应用测试仅访问测试进程资源；当前 WindowServer 保持本次启动 PID 198，没有再注销。

## 安装与下一次启动

surface_fix_admin.py 只在磁盘安装 XeGfxAccel.kext 与 XeMTLDriver.bundle，更新 AuxKC，保存旧二进制/旧 AuxKC/旧主收据；没有实时 unload/load，不写显示开关，不重启。原 NVIDIA、Intel 核心、AGDC、EFI 保持原收据校验。

修正版加速器 UUID：6A6FBD66-38DA-358C-AE55-7FE8A163AE17。旧的运行中加速器 UUID EF249C0A-C56D-3D4D-B9FF-CCA0C8D0F5D8 在下次手动重启前继续有效。

安装成功后，下次重启仍默认走 Hybrid 固件显示。按已有 probe_hybrid_stage2.py、probe_hybrid_engines.py 恢复 headless 阶段 1–7，再以管理员运行 probe_surface_fix.py。该脚本严格要求新 UUID、stage 7、无桌面/IOP/AGDC、没有准入名单；只测试内存描述符，并在 finally 中关闭 aperture 开关。它验证的是描述符工厂和内核 CPU 映射，不等于完整 IOAccelerator 用户映射或 WindowServer 桌面已成功。

本轮不会自动重试注销。只有新内核映射测试通过且另行准备好桌面验证方案后，才考虑图形会话切换。

## 回退层次

最新 surface-fix 安装成功后，先使用 surface_fix_admin.py rollback 加上该次 /Library/XeGfxTrial/surface-fix-时间 备份路径，恢复旧加速器、旧插件、旧 AuxKC 和主收据。该操作只改磁盘，手动重启后使用旧代码。

再需要回到显示接口安装前时，才能运行原 trial_display_admin.py rollback /Library/XeGfxTrial/display-20261023-230519。后者已加保护，若仍装着 surface-fix 用户态层，会拒绝跳过新层回退。不要热卸载显示对象。

Hybrid 无法进入桌面时，先切 BIOS Discrete；1401 中仍保留原有恢复材料。新层恢复入口和实际备份路径在 installation.log/系统收据中记录。

## 本次实际安装与恢复入口

备份：/Library/XeGfxTrial/surface-fix-20261024-004017

新 AuxKC SHA256：cd269a06bdc7d4be98298626f660747df43a3602ae46d6fed7fd756ef9db5bb9

最新层回退（管理员执行，只恢复磁盘，不自动重启）：

    /Library/Developer/CommandLineTools/usr/bin/python3 ~/Documents/Codex/2026-10-23/d/outputs/Intel-iGPU-275HX-20261023/surface_fix_admin.py rollback /Library/XeGfxTrial/surface-fix-20261024-004017
