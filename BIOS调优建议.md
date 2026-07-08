# AN515-58 (i7-12700H) BIOS 高级菜单调优建议

> 机型：Acer Nitro 5 AN515-58，CPU i7-12700H
> 现状：已用 SREP 解锁 Advanced 菜单、已禁用 E 核、功耗墙已 Unlock
> 目标场景：电竞网游，看重**高单核频率、低延迟、温度稳定、1% low 帧稳**

---

## 0. 动手前必读（重要）

1. **一次只改一项，改完存盘重启、进系统跑一会儿确认稳定，再改下一项。**
   多项一起改，出问题根本不知道是哪一项害的。
2. **改之前把原始值拍照/记下来**，尤其电压、功耗、锁相关的项，方便回退。
3. 电竞游戏（LoL / CS2）其实**吃 GPU 和单核多、吃多核少**。你已经关了 E 核，对这两款游戏反而是好事（消除大小核调度抖动）。所以调优重点不是"榨满多核功耗"，而是**降温 + 频率稳 + 延迟低**。
4. 每个菜单里名字可能因 BIOS 版本略有差异，找**关键词**即可。找不到完全同名的就找最接近的。

---

## 1. 先解开各种"锁"（否则后面很多都改不了）

这些一般在 `Advanced → CPU Configuration` 或 `Overclocking` / `Power & Performance` 菜单里。

| BIOS 英文项 | 中文含义 | 推荐值 | 为什么 |
|---|---|---|---|
| **CFG Lock** | MSR 0xE2 配置锁 | **Disabled** | 不解开，软件（ThrottleStop 等）无法写 CPU 配置寄存器，降压会失败 |
| **Overclocking Lock** | 超频锁 | **Disabled** | 锁着就没法调电压/倍频，改了也不生效 |
| **Overclocking Feature / Overclocking Support** | 超频功能总开关 | **Enabled** | 打开后才会出现电压/功耗调节子项 |

> 这三项是后面"降压"的前提。先把它们改好、重启确认能正常进系统，再往下走。

---

## 2. 降压（Undervolt）—— 电竞游戏最值得做的一步

**目的**：同样频率下电压更低 → 发热更少 → 不容易撞 100°C 温度墙 → 频率能持续拉得更高、帧数更稳。**这是纯赚，不是超频，不会不稳定到哪去，只要幅度别太激进。**

在 `Overclocking` / `CPU Power Management` 类菜单里找：

| BIOS 英文项 | 中文含义 | 推荐值 | 说明 |
|---|---|---|---|
| **CPU Core Voltage Mode** | 核心电压模式 | **Adaptive** 或 **Offset** | 选带 Offset（偏移）的模式 |
| **CPU Core Voltage Offset** | 核心电压偏移 | **先 -50 mV**，稳了再试 -75 / -100 mV | 负值＝降压。**一步 25mV 往下试**，别一上来 -150mV |
| **CPU Core Voltage Offset Prefix / Sign** | 偏移正负号 | **Minus (-)** | 有些 BIOS 正负号是单独一项，别设成加号变成加压 |
| **CPU Cache/Ring Voltage Offset** | 缓存(Ring)电压偏移 | 跟核心**同样的值** | 12 代建议 Core 和 Cache 降一样多，否则可能死机 |

**降压幅度怎么定**：
- i7-12700H 体质普遍能吃 **-80 ～ -125 mV**，但每颗芯片不同。
- 保守起步 **-50mV** → 重启 → 跑 10 分钟游戏或 Cinebench → 不蓝屏不花屏 → 再降 25mV → 重复。
- 一旦出现**死机 / 蓝屏 / 花屏 / 游戏崩溃**，说明降过头了，往回加 25mV 就是你的稳定值。

> ⚠️ 降压过头的表现是"系统突然重启或蓝屏"，**不会烧硬件**，别怕，退回上一档即可。

---

## 3. 功耗墙（Power Limit）—— 在 BIOS 里定死，解决你"100W 自动重置"的问题

你之前在软件里设的 100W 被后台程序覆盖了。**在 BIOS 里设是开机就写死的，最稳。**

在 `Power & Performance` / `CPU Power Management` 里找：

| BIOS 英文项 | 中文含义 | 推荐值（LoL/CS2） | 说明 |
|---|---|---|---|
| **Power Limit 1 (PL1) / Long Duration** | 长时功耗墙 | **65 ～ 75 W** | 电竞游戏根本吃不到 100W。PL1 定这个区间足够满帧，还更凉更安静 |
| **Power Limit 1 Time Window** | PL1 时间窗口 | 28 ～ 128 s | 保持默认即可 |
| **Power Limit 2 (PL2) / Short Duration** | 短时爆发功耗墙 | **90 ～ 100 W** | 加载、开局瞬间爆发用，短时间不影响温度 |
| **IccMax / Core Current Limit** | 核心电流上限 | 保持默认 | 除非撞电流墙，否则别乱动 |

> 说明：跑 LoL/CS2 时你去看 HWiNFO，CPU Package Power 大概率就 30～55W 徘徊，**根本用不满 100W**。所以你追求的从来不是"更高功耗"，而是"温度低、频率不掉"。PL1 设 65~75W 是甜点，既不撞墙也不烤鸡。

**改完之后**：进系统开 HWiNFO 看 PL1 是不是稳在你设的值、几分钟后没被弹回默认，就说明成功锁住了。

---

## 4. 阻止后台程序覆盖你的设置

导致"设了又被重置"的元凶，在 BIOS 里也能关：

| BIOS 英文项 | 中文含义 | 推荐值 | 说明 |
|---|---|---|---|
| **Intel Dynamic Tuning / DPTF** | Intel 动态调优 | **Disabled** | 最常见的"偷偷改回功耗墙"的元凶，关掉后你的 PL 设置才稳 |

> 关掉 DPTF 后，如果发现风扇策略变了，再配合 NitroSense 或后面的 ThrottleStop 兜底即可。

---

## 5. 延迟与频率相关（电竞向，锦上添花）

这些对 LoL/CS2 的**帧稳定性/输入延迟**有细微帮助，收益不如降压大，可选：

| BIOS 英文项 | 中文含义 | 推荐值 | 说明 |
|---|---|---|---|
| **Intel SpeedShift / HWP** | 硬件调频 | **Enabled** | 保持开启，让 CPU 快速升频，反应更跟手 |
| **Intel SpeedStep (EIST)** | 传统调频 | **Enabled** | 保持默认开启 |
| **C-States / Package C State Limit** | 深度休眠 | **保持 Enabled** | 笔记本别关，关了空载发热高、续航差，对这两款游戏帧数没实际提升 |
| **Turbo Boost / Turbo Mode** | 睿频 | **Enabled** | 必须开，关了就锁基频了 |
| **Race To Halt (RTH)** | 快跑快睡节能 | 可保持默认 | 影响很小 |

> 网上有人说"关 C-States 降延迟"——那是台式机极限电竞的玩法，笔记本上关了弊大于利（温度飙、续航崩），LoL/CS2 这种负载**感知不到差别**，不建议关。

---

## 6. 内存相关（谨慎，容易黑屏）

| BIOS 英文项 | 中文含义 | 推荐值 | 说明 |
|---|---|---|---|
| **Memory / DRAM Frequency** | 内存频率 | **保持默认（Auto）** | 笔记本 SODIMM 超频容易开不了机，收益又小，**新手别碰** |
| **XMP** | 内存超频档案 | 有就开 XMP，没有就默认 | 若原厂就有 XMP 档位可以开，但拿不准就别动 |

> ⚠️ 内存超频是最容易"改完开不了机"的重灾区。你目标是玩游戏不是跑分，**这一节建议整个跳过**。

---

## 7. 推荐的操作顺序（照这个流程走）

1. 解锁：`CFG Lock` / `Overclocking Lock` → Disabled，`Overclocking Feature` → Enabled → 存盘重启，确认能正常进系统。
2. 定功耗墙：PL1=70W、PL2=95W → 重启 → HWiNFO 确认稳住不弹回。
3. 关 DPTF：`Intel Dynamic Tuning` → Disabled → 重启确认功耗墙依然稳。
4. 降压：Core / Cache Offset 从 **-50mV** 起步，每次 -25mV，游戏跑稳了再往下，直到出错回退一档。
5. 确认 SpeedShift/Turbo 开着、C-States 别关。
6. 内存那节直接跳过。

---

## 8. 出问题怎么办（回退 / 救砖）

- **改完开不了机 / 黑屏**：断电，扣电池（或按开机键放电），一般会触发 BIOS 用默认值重启；不行就找 BIOS `Load Setup Defaults / F9` 恢复默认。
- **系统里蓝屏、花屏、游戏崩**：多半是**降压过头**，进 BIOS 把 Voltage Offset 往回加 25mV。
- **实在起不来**：了解 Insyde BIOS Crisis Recovery（危机恢复）流程再动手——所以第 0 节才强调**一次只改一项**，这样你永远知道是刚改的那项害的。

---

## 附：验证效果的方法

装 **HWiNFO64**，玩 20 分钟 LoL 或 CS2，重点看：
- **CPU Package Temperature**：降压后峰值温度应比之前低几度到十几度，不再长期贴 100°C。
- **CPU Package Power**：确认 PL1 稳在你设的值。
- **Performance Limit Reasons**（HWiNFO 里）：如果不再频繁显示 `Thermal`（温度墙），说明降压见效了。
- 游戏里看 **1% low 帧**：温度稳了之后，掉帧卡顿应该更少。
