# Microduck Replica BOM

非官方的 [Pollen Robotics / Hugging Face **Microduck**](https://github.com/pollen-robotics/microduck) 硬件复刻选型清单。

## 背景

Microduck 官方只开源了软件（机器人"大脑"，Rust 编写），**机械结构、PCB、正式 BOM 从未公开**。本项目的做法是：

1. 逐行挖掘官方 `pollen-robotics/microduck` 软件源码的注释/常量/测试用例，反推硬件规格的"实锤"证据（协议、总线、传感器型号、分辨率、ID 分配等）
2. 交叉参考官方唯一公开的硬件仓库 [`pollen-robotics/elec_RPI_Robot_HAT`](https://github.com/pollen-robotics/elec_RPI_Robot_HAT)（扩展板，KiCad+PCB+BOM+Gerber 全公开）
3. 参考同类社区复刻项目 [`JoyandAI/OpenMicroDuck`](https://github.com/JoyandAI/OpenMicroDuck) 已做的选型对照/证据分级方法论
4. 对每个部件标注证据等级（官方实锤 / 源码还原 / 社区还原 / 纯推测），选出目前能在淘宝/1688 等渠道买到的、性能最接近原版的替代件

详细清单见 **[BOM.md](./BOM.md)**。

## 当前状态

- 预算：约 ¥2742（不含结构件/3D打印/可选头部IMU），加头部IMU约¥2822
- 已确认/已选定的关键部件：
  - 主控：Radxa Zero 3W（官方SKU实锤）
  - 扩展HAT：直接照搬官方开源 `elec_RPI_Robot_HAT`，**分阶段populate**：Phase1(电源+TTL总线+40Pin+Qwiic，全为可手工焊封装)先做；RS485收发(HD-1910用不上)、板载BMI088 IMU(用途未查清)、音频(麦克风/编解码/功放)留到Phase2
  - 舵机：飞特 HD-1910-C001 ×15（因原版 Dynamixel XL330-M288-T 被炒到¥1200+/个而替换），数据接口实为Molex 5264 2.5mm(非2.0mm)，与HAT的JST-EH 2.5mm同间距
  - 机身/头部IMU转接：ESP32-C3 SuperMini + GY-BNO085（自制飞特从机协议桥接，本项目风险最高的部分）
  - ToF：VL53L8CX Qwiic 模块
  - 摄像头：Radxa 官方 Camera 8M 219 (IMX219)
  - 手柄：需支持 6 轴陀螺仪的 Switch Pro 兼容手柄（源码证实"晃手柄=鸭子转头"用到手柄自带IMU）
- 仍需自行开发（不在 BOM 预算内，属于软件/固件工作量）：
  1. ESP32-C3 上实现飞特（Feetech）从机协议，伪装成总线 ID 200 的 IMU
  2. `duck-control/bus.rs` 驱动层从 Dynamixel Protocol 2.0 改写为飞特协议
  3. HD-1910-C001 ↔ HAT 之间的同间距(2.5mm)换壳转接线材
- **HAT Phase2（先放一放，等Phase1跑通再考虑）**：
  1. RS485总线(SIT3088E)是否真的完全用不上，需要实际组装验证后再拍板要不要补
  2. 板载BMI088 IMU的软件角色未查清（跟总线上伪装ID200的IMU是两条独立通路），需要进一步翻`duck-control`源码或社区文档
  3. 音频链路(MEMS麦克风+TLV320AIC3104编解码+PAM8406D功放)何时补齐，取决于是否需要duck发声/收音功能

## 目录

- [`BOM.md`](./BOM.md) — 逐项选型表格，含原版规格证据、替代件参数、单价、选型理由
- 机械结构（外壳/连杆/关节支架）未包含在本项目，原版结构从未开源，如需复刻需另行测绘/建模

## 工作记录（Copilot CLI 会话）

- BOM 选型/HAT 拆解/采购状态跟踪：session `66c1e73f-6e35-42da-b926-9faad19c7184`（GitHub Copilot CLI），到 [`pollen-robotics/microduck_rl`](https://github.com/pollen-robotics/microduck_rl) 训练策略部分的调研由该会话末尾发起，后续训练相关工作建议另开新会话跟踪。

## 参考来源

- https://github.com/pollen-robotics/microduck — 官方软件源码（唯一的官方一手依据）
- https://github.com/pollen-robotics/elec_RPI_Robot_HAT — 官方唯一开源的硬件（扩展HAT）
- https://github.com/JoyandAI/OpenMicroDuck — 社区开源复刻项目，舵机/主控选型对照的重要参考
- https://deepwiki.com/pollen-robotics/rustypot — 舵机驱动库

## 免责声明

本项目与 Pollen Robotics / Hugging Face 官方无关，所有硬件选型均基于公开源码/文档的推断与社区还原，未经官方验证，仅供爱好者自行复刻参考。
