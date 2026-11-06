---
title: "RaC: Robot Learning for Long-Horizon Tasks by Scaling Recovery and Correction"
source: "https://arxiv.org/abs/2509.07953v1"
authors: ["Zheyuan Hu","Robyn Wu","Naveen Enock","Jasmine Li","Riya Kadakia","Zackory Erickson","Aviral Kumar"]
year: "2025"
status: preliminary
pdf_source: "https://arxiv.org/pdf/2509.07953v1"
pdf_version: "arXiv 2509.07953v1"
pdf_pages: 26
downloaded: 2026-11-06
category: "人工干预与纠正"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# RaC

**原文标题：** RaC: Robot Learning for Long-Horizon Tasks by Scaling Recovery and Correction

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/RaC.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2509.07953v1)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：人工接管时，应该示范怎样的一段恢复和纠正，而不只是帮机器人完成任务？**

## 问题与核心思路

RaC 把恢复和纠正作为需要专门采集的技能。在示范预训练之后，通过人类干预改善数据组成，保持模仿学习目标不变。[论文](https://arxiv.org/abs/2509.07953v1)

## 方法与数据

- **先恢复，再纠正**：先回到熟悉、可继续操作的状态，再完成当前子任务。
- **干预后结束 episode**：避免继续花预算采集人已修正前序状态后的后续子任务。
- 原文算法区分完整自主成功轨迹与干预轨迹；后者只将人控 transitions 加入对应新数据，需在精读时核对混合和预算口径。[原文 §4](https://arxiv.org/html/2509.07953v1)

## 实验与边界

实验包括挂衬衫、密封盒盖、打包外卖盒及仿真装配，比较完整示范与 HG-DAgger 风格采集，并研究两条采集规则。作者报告数据效率优势，但其“结束 episode”规则是特定长时序学习设定下的设计，不是所有项目的通用要求。[项目](https://rac-scaling-robot.github.io/)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：区分“回到可抓位置”和“完成下一次有效抓取”，避免把反复尝试全部当成同一种恢复。
- **Opendoor**：先划分接近、抓把手、拉门等阶段，再讨论接管段结束在哪里。不同阶段的失败不应只汇总成整条成功／失败。

## 精读问题

- [ ] recovery 与 correction 的边界怎样操作化？
- [ ] 终止规则如何改变后续子任务的状态分布与采集预算？
- [ ] 自主成功与人工数据的比例、损失和采样方式是什么？
- [ ] 等时间与等样本的对比是否给出一致结论？

## 关联

[[机器人学习的数据策略/论文/HG-DAgger|HG-DAgger]]关注接管；[[机器人学习的数据策略/论文/Mulligan|Mulligan]]还决定下一轮从哪里开始。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
