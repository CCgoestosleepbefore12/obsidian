---
title: "Robotic Data Inoculation"
source: "https://arxiv.org/abs/2610.39054v1"
authors: ["Qiyang Li","Zhiyuan Zhou","Charles Xu","Sergey Levine"]
year: "2026"
status: preliminary
pdf_source: "https://arxiv.org/pdf/2610.39054v1"
pdf_version: "arXiv 2610.39054v1"
pdf_pages: 16
downloaded: 2026-11-06
category: "交叉方法"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# Robotic Data Inoculation

**原文标题：** Robotic Data Inoculation

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/Robotic Data Inoculation.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2610.39054v1)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：能否用在线 RL 改变采集过程，主动获得对下游学习更有价值的恢复数据？**

## 问题与核心思路

DI 让人持续遥操，同时在线训练一个残差策略改变执行动作；人对偏差做出纠正，形成更丰富的恢复数据。作者将其用于下游 BC 和 offline RL。[论文](https://arxiv.org/abs/2610.39054v1)

## 方法与数据

- 执行动作为人的指令加有界残差，残差策略观察当前状态与人的指令。
- critic 评价实际扰动后的动作；残差策略优化任务奖励，并可加入残差幅度惩罚。
- 一个同步学习的 BC flow 模型近似人类指令，用于 TD 目标构造。
- 优化目标是协助任务完成，不是最大化失败；“学习中的偏差引出有用恢复”是作者给出的机制解释。[原文 §4，式2–4](https://arxiv.org/html/2610.39054v1)

## 实验与边界

论文在三个真实任务上比较采集方案，再分别训练 BC 与离线 RL 策略。精读应检查各任务、种子和预算下的收益，而不能把所有改进都解释为已证明“复现了下游策略的错误”。[实验原文](https://arxiv.org/html/2610.39054v1)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：先建立普通示范、针对性恢复和人工设计扰动的可比较数据，再评估在线学习残差的必要性。
- **Opendoor**：先确认在线奖励、控制接口及人机同时参与的记录条件；现有 PI0.5 数据训练链并不自动包含这些能力。
- 工程上应区分人类原始指令、残差、实际执行动作、反馈及后续纠正，之后按训练模块选择数据。

## 精读问题

- [ ] 下游 BC、离线 RL 与残差学习分别用哪些动作字段？
- [ ] 残差幅度和奖励选择对采集效率、恢复覆盖有多敏感？
- [ ] 哪些证据能把数据覆盖收益与辅助执行收益区分开？
- [ ] 若只用现有 PI0.5 做示范微调，论文中哪些模块可以保留，哪些假设会变化？

## 关联

对照 [[机器人学习的数据策略/论文/DART|DART]] 的噪声、[[机器人学习的数据策略/论文/ADC|ADC]] 的人为扰动，以及 [[机器人学习的数据策略/论文/Residual Copilot|Residual Copilot]] 的辅助控制。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
