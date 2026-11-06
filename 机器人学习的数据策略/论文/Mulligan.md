---
title: "Mulligan: Performance-Guided Data Collection for Efficient On-Robot Learning"
source: "https://arxiv.org/abs/2610.05882v1"
authors: ["Lars Ankile","Perry Dong","Rohan Bhowmik","Aneesh Muppidi","David D. Yuan","Shuran Song","Chelsea Finn"]
year: "2026"
status: preliminary
pdf_source: "https://export.arxiv.org/pdf/2610.05882v1"
pdf_version: "arXiv 2610.05882v1"
pdf_pages: 65
downloaded: 2026-11-06
category: "人工干预与纠正"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# Mulligan

**原文标题：** Mulligan: Performance-Guided Data Collection for Efficient On-Robot Learning

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/Mulligan.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2610.05882v1)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：在固定采集预算下，下一轮应该从哪些初始状态开始，补恢复还是补正确完整示范？**

## 问题与核心思路

策略改进后，失败可能集中在少数初始状态。Mulligan 根据实际表现选择下一轮采集起点，兼顾已知失败与尚未覆盖的区域。[论文](https://arxiv.org/abs/2610.05882v1)

## 方法与数据

- **Failure promotion**：优先回访已观察到失败的初始状态。
- **Coverage fill**：其余预算用于未充分覆盖的初始状态。
- **Counterfactual demonstrations**：纠正若只展示如何恢复，必要时从相同初始状态重新示范完整正确过程。
- 学习端将 actor 与 critic 分开：actor 使用成功 episode 中的人控数据；critic 利用全部数据，包括失败。价值函数还用于部署时选择动作块。[原文 §3](https://arxiv.org/html/2610.05882v1)

## 实验与边界

作者比较采集策略及学习方法，完整 HiL-IDQL＋Mulligan 在三个实机任务中报告相对基线提高10–34个百分点。这个数字属于组合方法，不能全部归因于初始状态选择；相同 episode 数也未必意味着相同人工时间。[项目](https://mulligan.page/)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：保存胶条初始摆位与失败阶段，区分“纠正夹空”与“从一开始正确接近”的监督。
- **Opendoor**：记录导航到位后的机器人／门相对条件，评估下一批是否应覆盖特定起点；评测集与补采选择过程应可追溯。
- 可以单独借鉴采集场景选择；完整价值重排需要额外学习组件。

## 精读问题

- [ ] 如何定义初始状态距离和未覆盖区域？
- [ ] 失败起点何时回采，何时需要完整反事实示范？
- [ ] 采集策略与 critic 的收益能否从消融中分离？
- [ ] 评测状态用于下一轮选点后，后续评估怎样保持独立？

## 关联

[[机器人学习的数据策略/论文/RaC|RaC]]规定如何干预；[[机器人学习的数据策略/论文/RECAP|RECAP]]采用另一种价值与策略学习方式。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
