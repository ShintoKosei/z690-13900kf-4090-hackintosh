# 13900KF-Z690-4090-EFI

![macOS](https://img.shields.io/badge/macOS-26%20Tahoe-000000?logo=apple&logoColor=white)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.9-4B8BBE)
![CPU](https://img.shields.io/badge/CPU-i9--13900KF-0071C5?logo=intel&logoColor=white)
![GPU](https://img.shields.io/badge/GPU-RTX%204090-76B900?logo=nvidia&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

> 适用于 **i9-13900KF + MSI Z690 FORCE WIFI + RTX 4090** 的 macOS 26 (Tahoe) OpenCore EFI 配置。
>
> 基于 [1401](https://github.com/nullmoth/1401) v2.5.3 生成，已按 v2.5.4 的 `HideVerbose` 修复要求补齐。

---

## ⚠️ 使用前必读

**本仓库的序列号、MLB、UUID、ROM 均为占位符，直接使用无法登录 App Store / iCloud。**

使用前必须自己生成三码并填入 `OC/config.plist`：

| 字段 | 当前占位值 |
|---|---|
| `SystemSerialNumber` | `C02XXXXXXXXXX` |
| `MLB` | `C02XXXXXXXXXXXXXXX` |
| `SystemUUID` | `00000000-0000-0000-0000-000000000000` |
| `ROM` | *(空)* |

**生成工具：**

- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) — OpenCore 官方配套，Windows / macOS 通用
- [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools) — 图形界面，可直接编辑 config.plist
- `macserial` — macOS 命令行

机型请保持 **`MacPro7,1`** 不变。

---

## 硬件配置

| 部件 | 型号 | 状态 |
|---|---|---|
| **CPU** | Intel Core i9-13900KF | ✅ 原生支持（含大小核），CPUFriend 调优后单核 5.48 GHz |
| **主板** | MSI MPG Z690 FORCE WIFI (MS-7D30) | ✅ |
| **显卡** | NVIDIA GeForce RTX 4090 (Ada) | ⚠️ 需 NullMoth 驱动；硬解可用、硬编不可用 |
| **内存** | 128 GB DDR5 | ✅ |
| **有线网卡** | Intel I225-V (2.5GbE) | ✅ AppleIGC |
| **无线网卡** | Intel Wi-Fi 6E AX211 | ✅ AirportItlwm 原生（无需 HeliPort），无 AirDrop |
| **蓝牙** | Intel AX211 板载 | ✅ Intel 三件套 2.5.0 |
| **存储** | NVMe SSD | ✅ NVMeFix |

> **外接的 Realtek USB 蓝牙适配器已停用**——板载 Intel 蓝牙与它不能共存，用板载时需要拔掉。

---

## 快速开始

```bash
# 1. 克隆或下载本仓库
git clone https://github.com/ShintoKosei/z690-13900kf-4090-hackintosh.git

# 2. 生成三码并填入 OC/config.plist
#    用 GenSMBIOS 或 OCAuxiliaryTools，机型选 MacPro7,1

# 3. 把整个 EFI 文件夹复制到 U 盘 / 硬盘的 ESP 分区（FAT32，≥200MB）
```

**ESP 分区在 Windows 下默认不显示**，用 [DiskGenius](https://www.diskgenius.cn/) 或 `mountvol X: /S` 访问。

> 💡 **装完就能用，不需要额外安装任何 App**：Wi-Fi 由 `AirportItlwm` 注册为原生 AirPort 端口，蓝牙由 Intel 三件套驱动，系统设置里直接管理。
>
> ⚠️ **用板载蓝牙前请拔掉 Realtek USB 蓝牙适配器**（两个蓝牙栈不能共存）。

---

## BIOS 设置

| 设置项 | 值 | 说明 |
|---|---|---|
| **Secure Boot** | Disabled | 必须 |
| **CSM** | Disabled | 纯 UEFI 模式 |
| **Above 4G Decoding** | Enabled | 大显存必需 |
| **XHCI Hand-off** | Enabled | USB 必需 |
| **Resizable BAR** | Disabled | NVIDIA 卡建议关闭 |
| **CFG Lock** | Disabled | 如可关闭 |
| **VT-d** | Disabled | 或配合 `DisableIoMapper` |

---

## 目录结构

```
EFI/
├── BOOT/
│   └── BOOTx64.efi              OpenCore 引导器
└── OC/
    ├── ACPI/                    SSDT 补丁
    ├── Drivers/                 UEFI 驱动
    │   ├── apfs_aligned.efi
    │   ├── HfsPlus.efi
    │   ├── OpenCanopy.efi       图形主题支持
    │   ├── OpenRuntime.efi
    │   └── ResetNvramEntry.efi
    ├── Kexts/                   19 个启用的内核扩展（另 2 个保留但禁用）
    ├── Resources/               主题资源
    │   └── Image/
    │       ├── Acidanthera/     内置主题
    │       └── chris1111/       Flavours-OSX26 主题
    ├── Tools/
    │   └── NullMothSafe.efi     NVIDIA 驱动救援工具
    ├── OpenCore.efi
    └── config.plist
```

---

## 已启用的 Kext

<details>
<summary>点击展开 19 个 kext 及作用</summary>

| Kext | 作用 |
|---|---|
| `Lilu.kext` | 补丁框架（其他 kext 的依赖） |
| `AMFIPass.kext` | 绕过 AMFI 签名检查 |
| `AppleIGC.kext` | Intel I225-V 2.5G 有线网卡 |
| `BlueToolFixup.kext` | 蓝牙通用补丁（Monterey+） |
| `IntelBTPatcher.kext` | Intel 蓝牙 macOS 12+ 兼容补丁（2.5.0） |
| `IntelBluetoothFirmware.kext` | Intel 蓝牙固件上载（2.5.0） |
| `AirportItlwm.kext` | **Intel Wi-Fi 原生驱动**（laobamac run53，支持 CNVi/AX211） |
| `CpuTopologyRebuild.kext` | 大小核拓扑修正 |
| `CPUFriend.kext` | CPU 电源管理数据注入（1.3.1） |
| `CPUFriendDataProvider.kext` | 自定义频率向量 / 电源策略（针对 13900KF 调优） |
| `NVMeFix.kext` | NVMe 电源管理（APST） |
| `RestrictEvents.kext` | 屏蔽不需要的系统事件 |
| `VirtualSMC.kext` | SMC 模拟核心 |
| `SMCProcessor.kext` | CPU 传感器 |
| `SMCSuperIO.kext` | 主板传感器 |
| `USBToolBox.kext` | USB 端口映射框架 |
| `UTBDefault.kext` | USB 端口默认配置 |
| `WhateverGreen.kext` | 显卡补丁 |
| `XHCI-unsupported.kext` | 12/13 代 PCH XHCI 支持 |

</details>

**另有 2 个保留但已禁用的 kext：**

| Kext | 禁用原因 |
|---|---|
| `RealtekBluetoothFirmware.kext` | 外接 Realtek USB 蓝牙；改用板载 Intel 蓝牙后停用 |
| `itlwm.kext` | 旧 Wi-Fi 方案（需 HeliPort）；改用 `AirportItlwm` 后停用 |

> ⚠️ `itlwm` 与 `AirportItlwm` 是两套 Wi-Fi 栈，**同时启用会冲突**，只留一个。
>
> ⚠️ 使用板载 Intel 蓝牙时，**必须拔掉 Realtek USB 蓝牙适配器**（[官方 FAQ](https://openintelwireless.github.io/IntelBluetoothFirmware/FAQ.html)：Intel 蓝牙不能与其他蓝牙共存）。

---

## 已知限制

### 🔵 无线网卡（AX211）

- ✅ **原生 Wi-Fi**：`AirportItlwm.kext` 注册为 AirPort 端口，系统设置直接管理，**不需要 HeliPort**
- ✅ 开机自动连接、菜单栏原生图标、CoreLocation 定位可用
- ❌ **AirDrop / 接力 / 随航不可用** —— Intel 蓝牙过不了 Apple 连续互通校验，这是物理限制
- ⚠️ 换驱动族后 `en` 序号可能重排，iMessage / FaceTime 可能要求重新登录一次

> 如果你在旧版本 macOS 上，仍可用 `itlwm.kext` + [HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases) 方案（把 `itlwm` 启用、`AirportItlwm` 禁用即可）。

### 🟢 NVIDIA 显卡（RTX 4090）

- 需要安装 **[NullMoth NVIDIA 驱动](https://github.com/nullmoth/1401/releases)**（本仓库不含驱动本体）
- 建议 **2.5.4 或更新**（修复了「仅 NVIDIA 卡桌面黑屏循环」）
- `Tools/NullMothSafe.efi` 在 OpenCore 菜单提供两个救援项：
  - `1401: Remove NVIDIA driver` — 驱动装崩了一键移除
  - `1401: Start once without NVIDIA driver` — 无驱动启动一次

**视频编解码实测：**

| 能力 | 状态 | 说明 |
|---|---|---|
| H.264 硬解 | ✅ 875 FPS | NVDEC 正常工作 |
| HEVC 硬解 | ✅ 1381 FPS | NVDEC 正常工作 |
| H.264 硬编 | ❌ `-12912` | macOS 不支持 NVENC |
| HEVC 硬编 | ❌ `-12912` | 同上 |
| H.264 软编 | ⚠️ 116 FPS | 可用 |
| HEVC 软编 | ⚠️ 4.2 FPS | 基本不可用 |

> **编码不可用是硬件限制，无解**：macOS 的 VideoToolbox 只支持 Intel QuickSync / AMD VCE / Apple Silicon。NVIDIA NVENC 从未被 macOS 支持，且 13900KF 是无核显的 KF 型号。
>
> 需要硬件编码的话，加一块 AMD 卡（RX 6600/6800）最划算——顺带还能补上 AirDrop。

### 🔊 音频

- macOS 26 移除了 AppleHDA，板载声卡可能需要 **VoodooHDA + KDK**

---

## 常见问题

<details>
<summary><b>启动卡在 <code>AppleUSBXHCIPCI::start failed: 0xe00002c7</code></b></summary>

**不要添加 `Kernel > Block` 屏蔽 `com.apple.iokit.IONDRVSupport`。**

虽然这条注释写着「防止 IONDRVFramebuffer 抢占显示索引 0」，但实测会连带导致 USB 3.x 控制器初始化失败（`kIOReturnUnsupported`），USB 设备全部失效。

如果显示正常，这项**永久不需要**。

</details>

<details>
<summary><b>卡在 <code>DMTF_ERR: code = 9</code>，opendirectoryd 崩溃</b></summary>

SMBIOS 机型与内存槽数量不匹配。

- `MacPro7,1` 需要 `SystemMemoryStatus = Upgradable`（OpenCore 1.0.9 不支持 `Filled`）
- 或改用 `iMac20,2` + `UpdateSMBIOSMode = Create`

</details>

<details>
<summary><b>安装器提示「Mac 需要更新固件才能安装到此宗卷」</b></summary>

检查 `PlatformInfo > Generic`：

```xml
<key>MaxBIOSVersion</key><true/>     <!-- 绕过 BIOS 版本检查 -->
<key>AdviseFeatures</key><true/>     <!-- 启用 APFS 启动支持 -->
```

同时确认 `UpdateSMBIOSMode = Custom`（避免固件在 ExitBootServices 覆盖 SMBIOS）。

</details>

<details>
<summary><b>启动菜单只有文字，没有主题</b></summary>

确认三项：

1. `UEFI > Drivers` 中有 `OpenCanopy.efi` 且 `Enabled = true`
2. `Misc > Boot > PickerMode = External`
3. `Misc > Boot > PickerAttributes = 145`（含 128 = 启用自定义主题位）

</details>

<details>
<summary><b>主题显示但图标不对/变形</b></summary>

`PickerVariant` 格式为 `作者名\主题名`，本仓库为：

```
chris1111\Flavours-OSX26
```

注意是反斜杠，不是斜杠。

</details>

---

## 配置摘要

```yaml
机型:            MacPro7,1
SMBIOS 模式:     Create
内存状态:        Upgradable
OpenCore:        1.0.9 (DBG-109)
主题:            chris1111\Flavours-OSX26 (PickerAttributes=145)
分辨率:          Max (自动 4K)
Kernel > Block:  0 项
Kexts:           19 启用 / 21 总
Drivers:         5 个
Wi-Fi:           AirportItlwm 2.4.0（原生，无需 HeliPort）
蓝牙:            Intel 三件套（固件 2.5.0）
CPU 电源:        CPUFriend + 调优数据（单核实测 5.48 GHz）
```

---

## 更新日志

### 2026-10-28
- **Wi-Fi 换原生方案**：新增 `AirportItlwm` 2.4.0（laobamac run53），支持 CNVi/AX211，不再需要 HeliPort
- **蓝牙升级**：`IntelBluetoothFirmware` / `IntelBTPatcher` 2.4.0 → 2.5.0，启用板载 Intel 蓝牙，禁用 `RealtekBluetoothFirmware`
- **新增 CPUFriend**：`CPUFriend` 1.3.1 + 调优的 `CPUFriendDataProvider`（`perf-bias`、`iocs_*`、`tlvl` 等五项）
- **睿频实测**：单核 4.48 → **5.48 GHz**，多核 4.21 → 5.22 GHz
- 禁用 `itlwm`（与 AirportItlwm 冲突）

### 2026-10-24
- 初始发布
- 补齐 `OpenCanopy.efi` 的 `HideVerbose` 键（对应 1401 v2.5.4 修复）
- 三码清空为占位符
- 剥离全部 Authenticode 签名

---

## 致谢

- [Acidanthera](https://github.com/acidanthera) — OpenCore、Lilu、VirtualSMC、WhateverGreen、NVMeFix、RestrictEvents
- [NullMoth](https://github.com/nullmoth/1401) — 1401 构建工具、NVIDIA 驱动
- [OpenIntelWireless](https://github.com/OpenIntelWireless) — itlwm、HeliPort、IntelBluetoothFirmware
- [chris1111](https://github.com/chris1111/My-Simple-OC-Themes) — Flavours 主题
- [corpnewt](https://github.com/corpnewt) — GenSMBIOS、USBToolBox

---

## 免责声明

本项目仅供**相同硬件配置**参考学习。黑苹果涉及修改系统引导，存在数据丢失、硬件损坏、账号封禁等风险，**请自行备份并承担后果**。

不同 BIOS 版本、硬件批次可能需要微调。作者不对任何直接或间接损失负责。

**请勿将本配置用于商业用途。**
