# Acer Nitro 5 AN515-58 BIOS 解锁

解锁 Acer Nitro 5 **AN515-58** 隐藏的 **Advanced（高级）** BIOS 菜单，用于释放性能（超频/内存时序/风扇曲线/功耗墙等）。

## 适用机型

| 项目 | 值 |
|------|-----|
| 机型 | Acer Nitro 5 **AN515-58** |
| 主板代号 | `Jimny_ADH` |
| CPU | Intel Core **i7-12700H**（Alder Lake-H） |
| GPU | NVIDIA RTX 3060 (Laptop) |
| BIOS 类型 | Insyde H2O |
| 验证的 BIOS 版本 | **V1.04**（由 V2.19 降级而来） |

> 本仓库的 `SREP_Config.cfg` 已针对 **AN515-58 / Predator PH517-55** 的 Insyde BIOS 字节特征做过匹配，并用 **V1.04 官方镜像** 以及本机 **实时 SPI Flash dump** 双向核对过（53/70 特征命中，其余为版本差异项）。同版本机器可直接使用。

> ⚠️ 本仓库最初自带的 `SREP_Config.cfg` 其实是 **Predator Triton 300（AN515-57）** 的配置，装在 AN515-58 上几乎解不出菜单。现已替换为 AN515-58 专用配置。

## 工作原理

使用 [SmokelessRuntimeEFIPatcher (SREP)](https://github.com/SmokelessCPUv2/SmokelessRuntimeEFIPatcher)：从 U 盘启动，**在内存里** patch 掉 `SetupUtility` 的 `SuppressIf` 逻辑，把被隐藏（灰掉）的高级页面显示出来。

**它只改内存，不写入 BIOS Flash——重启即恢复，不会永久修改固件，相对安全。**

## 使用步骤

1. 准备一个 U 盘，格式化为 **FAT32 / GPT**（会清空 U 盘数据）。
2. 把本仓库的 `EFI/` 文件夹和 `SREP_Config.cfg` 复制到 U 盘 **根目录**：
   ```
   U盘根目录/
   ├── EFI/BOOT/BOOTX64.efi     ← 名字固定，不能改
   └── SREP_Config.cfg          ← 名字固定，不能改（硬编码在 .efi 里）
   ```
3. 关闭 Secure Boot（否则无法从 U 盘启动 SREP）：
   开机按 `F2` 进 BIOS > **Security** > 先 *Set Supervisor Password* ✔ > **Boot** > *Secure Boot: Disabled* ✔ > `F10` 保存退出。
4. 重启，从 U 盘启动（开机按 `F12` 选启动项）。
5. SREP 自动运行、打补丁，然后进入已解锁的 BIOS。Advanced 菜单出现。
6. 运行日志写在 U 盘的 `SREP.log`，可用于排查哪条补丁命中/未命中。

## 文件名说明

- `EFI/BOOT/BOOTX64.efi`：UEFI 后备启动约定的固定路径，**不能改名**。
- `SREP_Config.cfg`：文件名**硬编码**在 `BOOTX64.efi` 内部（可用 `strings` 看到 `Failed on Opening SREP_Config`），改成别的名字 SREP 会报打不开而直接跳过，**不能改名**。

## 本机性能背景（AN515-58 专属笔记）

- 性能瓶颈实测是 **EC / DPTF 的 ~80W 功耗墙**，不是温度问题（80W 时 CPU 仅 ~75°C，散热余量很大）。解锁 Advanced 后可在 DPTF / PL1 / PL2 相关项里尝试抬高功耗墙。
- 从 V2.19 降级到 V1.04 后，OS 会自动加载更新的微码（0x43b）覆盖 BIOS 里的旧微码（0x41b）。若要保持 V1.04 的微码，已在 GRUB 加了 `dis_ucode_ldr`（`/etc/default/grub`，改后执行 `sudo update-grub`）。
- **EC / ME 模块无需降级**：ME 有防回滚熔丝（不可逆）、且对性能无帮助；EC 降级风险大，功耗墙问题在解锁后的 DPTF 里解决更稳妥。

## ⚠️ 风险警告

解锁后暴露的很多项（内存时序/频率、手动电压、iGPU/PCIe 等）**改错可能直接变砖**。

- 一次只改一项，改前记住原值。
- 动手前先了解本机的 BIOS Crisis Recovery（应急刷写）流程。
- 后果自负。

## 相关工具

- [SmokelessRuntimeEFIPatcher](https://github.com/SmokelessCPUv2/SmokelessRuntimeEFIPatcher) — 内存 patch 引导器（本仓库 `EFI/BOOT/BOOTX64.efi`）
- 配置来源：[Maxinator500/SREP-Patches](https://github.com/Maxinator500/SREP-Patches) 中的 Acer AN515-58 (517-55) 配置

## 备注

配置通过 patch `SetupUtility` 与 `A01ODMDxeDriver` 模块来取消对 Advanced/隐藏页面的抑制。字节特征是针对特定 Insyde BIOS 编译版本匹配的，**务必使用与自己机型一致的配置**，其他 Acer 机型的配置见上游仓库 `Configs/Acer/`。
