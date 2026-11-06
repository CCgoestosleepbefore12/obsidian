---
title: "Efficient and Reliable Teleoperation through Real-to-Sim-to-Real Shared Autonomy"
source: "https://arxiv.org/abs/2603.17016v2"
authors: ["Shuo Sha","Yixuan Wang","Binghao Huang","Antonio Loquercio","Yunzhu Li"]
year: "2026"
status: preliminary
pdf_source: "https://arxiv.org/pdf/2603.17016v2"
pdf_version: "arXiv 2603.17016v2"
pdf_pages: 17
downloaded: 2026-11-06
category: "AI 辅助采集"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# Residual Copilot

**原文标题：** Efficient and Reliable Teleoperation through Real-to-Sim-to-Real Shared Autonomy

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/Residual Copilot.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2603.17016v2)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：辅助策略能否改善人采出来的数据，而不仅是让当次遥操更容易？**

## 问题与核心思路

Residual Copilot 用少量真实遥操数据建立 kNN 人类行为代理，在仿真中训练残差辅助策略，再部署到真实系统帮助人进行精细接触操作。[论文](https://arxiv.org/abs/2603.17016v2)

## 方法与数据

- 人类行为代理为仿真提供近似人的控制输入，避免训练辅助策略时始终需要真人参与。
- 用 RL 学习结合状态和人的指令进行低层修正的 copilot。
- 部署时人持续操控，辅助策略补充修正；系统还包含实际控制与状态估计环节。
- 辅助采集生成的数据再用于训练独立的模仿策略。[项目](https://residual-copilot.github.io/)

## 实验与边界

论文在螺母旋拧、齿轮啮合和插销任务中进行用户研究，并分别比较遥操表现与下游示范质量。数据质量实验包含匹配尝试数和匹配成功示范数的比较。仿真资产、人类代理与真实接触控制的适配仍是迁移条件。[原文 §IV-C](https://arxiv.org/html/2603.17016v2)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：先判断采集质量受人的精度还是机器人执行节奏限制；柔性胶条的仿真条件需要单独评估。
- **Opendoor**：检查接触操作中哪些修正可由辅助策略完成；不能将正常辅助与记录断流问题混为一谈。
- 需要分别评测人机采集的完成率、人工负担和下游自主策略效果。

## 精读问题

- [ ] kNN 人类代理输入哪些状态，数据不足时怎样泛化？
- [ ] 残差在哪个动作空间叠加，如何保留人的意图？
- [ ] 下游训练使用什么动作标签，和实际执行动作是否相同？
- [ ] 仿真建模与训练成本是否计入总体采集成本？

## 关联

[[机器人学习的数据策略/论文/Robotic Data Inoculation|Data Inoculation]]也采用共享控制，但在真实采集期间在线学习，并关注恢复覆盖。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
