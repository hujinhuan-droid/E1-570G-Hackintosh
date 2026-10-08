---
AIGC:
  ContentProducer: '001191110102MAD55U9H0F10002'
  ContentPropagator: '001191110102MAD55U9H0F10002'
  Label: '1'
  ProduceID: '745b974e-b2d9-4e0d-b804-6274ad471ae2'
  PropagateID: '745b974e-b2d9-4e0d-b804-6274ad471ae2'
  ReservedCode1: 'fe494ec0-29f8-4cf5-bc11-147008d65f8f'
  ReservedCode2: 'fe494ec0-29f8-4cf5-bc11-147008d65f8f'
---

# Acer Aspire E1-570G Hackintosh (OpenCore + macOS Monterey 12.7.6)

Acer Aspire E1-570G 笔记本的 OpenCore 黑苹果引导配置。**已实现全功能可日常使用**：显卡全速、声卡、有线网卡均正常。

## 硬件配置

| 部件 | 型号 | 状态 |
| ---- | ---- | ---- |
| CPU | Intel Core i3-3217U (Ivy Bridge) | ✅ |
| 内存 | 8GB DDR3 | ✅ |
| 核显 | Intel HD4000 | ✅ 已打补丁全加速（非 4MB） |
| 独显 | NVIDIA GT740M | ⏸️ 已禁用（`-wegnoegpu`） |
| 声卡 | Realtek ALC282 | ✅ `alcid=28` + HPET/IRQ 修复补丁 |
| 有线网卡 | Broadcom BCM57781 (14e4:16b3) | ✅ CatalinaBCM5701Ethernet 驱动 |
| 无线网卡 | Atheros AR9565 (168c:36) | ❌ macOS 无驱动，建议有线或 USB 无线网卡 |
| 屏幕 | 1366x768 | 屏幕有一条硬件竖线（本机故障） |

## 软件版本

- macOS Monterey 12.7.6
- OpenCore 0.9.8 (RELEASE)
- Lilu 1.7.2 / WhateverGreen 1.7.0 / VirtualSMC 1.3.7 / AppleALC 1.7.4 / VoodooPS2Controller 2.3.7
- CatalinaBCM5701Ethernet（有线网卡驱动，来自 OCLP）

## EFI 目录结构

```
EFI/
├── APPLE/                       # Apple 固件（保留原版）
├── BOOT/BOOTx64.efi
└── OC/
    ├── config.plist             # 主配置
    ├── OpenCore.efi
    ├── ACPI/                    # SSDT 补丁（EC/HPET/HDEF/Disable-DGPU 等）
    ├── Drivers/                 # OpenHfsPlus / OpenRuntime / 键盘鼠标驱动
    ├── Kexts/                   # 全套 kext 驱动
    └── Tools/                   # CleanNvram / OpenShell
```

## 安装步骤

> 以下步骤基于「已有 macOS 分区（Mojave 或空分区）+ Windows 双系统」的场景；全新安装从第 1 步开始。

### 1. 准备工具

- 8GB 以上 U 盘
- macOS Monterey 安装器（App Store 下载或 `installinstallmacos.py`）
- 下载 **Patch-HD4000-Monterey**（chris1111，用于 HD4000 显卡补丁）：https://github.com/chris1111/Patch-HD4000-Monterey

### 2. 制作安装 U 盘（Mac 下执行）

```bash
sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/YourUSB --nointeraction
```

### 3. 部署 EFI

- 将本仓库 `EFI/` 目录整个复制到安装 U 盘和硬盘 ESP 分区的 `EFI/`（覆盖旧引导）
- 硬盘 ESP 挂载（Windows 下管理员 CMD）：
  ```
  mountvol Z: /S
  ```

### 4. BIOS 设置

- 关闭 Secure Boot（CFG Lock 已通过 config 补丁处理）
- 启动顺序调整为 U 盘优先

### 5. 安装 macOS Monterey

- OpenCore 菜单选 U 盘的 macOS 安装器
- 安装完成后重启选硬盘的 macOS 卷进入
- 说明：EFI 默认使用 `MacBookPro12,1` 机型（使 Monterey 安装器接受硬件），序列号已内置

### 6. 显卡补丁（HD4000 全加速）

- 系统默认 HD4000 只有 4MB 显存，需打补丁
- 先确保 SIP 彻底关闭：本 EFI 已配置 `csr-active-config=EF0F0000`
- 首次配置后需在 OpenCore 菜单执行一次 **CleanNvram**
- 运行 **Patch-HD4000-Monterey** 应用 → 按提示打补丁 → 重启
- 验证：关于本机 → 显示卡，显存恢复正常

### 7. 声卡

- 已内置 AppleALC + 声卡 IRQ 补丁（HPET/IRQ 中断修复 + SSDT-HDEF），开机即有声音
- 参数：`alcid=28`（在 boot-args 与设备属性中保持一致）

### 8. 有线网卡

- 已内置 CatalinaBCM5701Ethernet 驱动 + 设备伪装（device-id=16b4）
- 插网线即可上网

## 注意事项

1. **SIP 必须彻底关闭**（打显卡补丁的前提）：`csr-active-config = EF0F0000`，改过 NVRAM 相关设置后务必 **Reset NVRAM（OpenCore 菜单 → CleanNvram）**
2. **不要改动系统原生驱动**：显卡补丁后若恢复（升级）系统，需重新打补丁
3. **WiFi 无解**：AR9565 在 macOS 没有驱动，请使用有线网卡或 USB 无线网卡
4. **时间不准**：若重启后时间漂移，建议更换主板纽扣电池（CR2032）
5. **SMBIOS 信息**：本仓库内置了可用的 MacBookPro12,1 序列号信息（仅为正常引导使用）。如需登录 iMessage/FaceTime，建议自行用 GenSMBIOS 生成专属序列号并替换 `PlatformInfo > Generic` 段
6. **双系统引导**：OpenCore 菜单中同时存在 Windows 与 macOS，选择即可切换
7. **显示器竖线**：本机屏幕有一条硬件竖线（拍摄图可见），不影响系统功能

## 常见问题排查

| 现象 | 处理 |
| ---- | ---- |
| 启动卡 EXITBS 或崩溃 | 确认 `AppleCpuPmCfgLock=true`、`-no_compat_check` 等 Kernel/ACPI 补丁完整 |
| 声音没有 | 确认 ACPI 补丁中的 IRQ 修复与 SSDT-HDEF 存在，且 `alcid` 与设备属性 `layout-id` 一致 |
| 网卡没有 | 确认 CatalinaBCM5701Ethernet.kext 被加载、device-id 伪装为 16b4 |
| 显存 4MB | 运行 Patch-HD4000-Monterey 并重启（需先 CleanNvram） |

## 致谢

- OpenCore / Dortania 教程
- chris1111 的 Patch-HD4000-Monterey 工具
- 网上各位黑苹果同机型（E1-471G）分享者的经验

> AI生成