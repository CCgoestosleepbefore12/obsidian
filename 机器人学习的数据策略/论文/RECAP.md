---
title: "π*0.6: a VLA That Learns From Experience"
source: "https://arxiv.org/abs/2511.14759v2"
authors: ["Physical Intelligence","Ali Amin et al."]
year: "2025"
status: preliminary
pdf_source: "https://arxiv.org/pdf/2511.14759v2"
pdf_version: "arXiv 2511.14759v2"
pdf_pages: 18
downloaded: 2026-11-06
category: "人工干预与纠正"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# RECAP

**原文标题：** π*0.6: a VLA That Learns From Experience

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/RECAP.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2511.14759v2)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：除了把纠正动作作为示范，怎样让自主成功和失败经验也参与改进 VLA？**

## 问题与核心思路

RECAP 的全称是 RL with Experience and Corrections via Advantage-conditioned Policies。它结合示范、自主执行和人工纠正，通过价值估计与优势条件化改进策略；采集纠正数据只是其中一部分。[论文](https://arxiv.org/abs/2511.14759v2)

## 方法与数据

- 先从多任务数据学习价值与策略，再通过真实机器人执行及专家接管收集新经验。
- 根据任务结果训练价值模型，以动作优势构造改进条件。
- 策略学习带条件的动作分布，执行时请求较高优势的行为。
- 失败经验能提供价值学习信号，不意味着把所有失败动作无区别地做成功示范 BC。[官方方法介绍](https://www.pi.website/blog/pistar06)

## 实验与边界

作者在折衣服、组装箱子和制作咖啡等任务中报告吞吐量及失败率改善。量化收益与具体任务和完整训练方法绑定，不能直接当作“只补接管数据”的收益。[论文](https://arxiv.org/abs/2511.14759v2)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：先把人控纠正、自主成功、自主失败区分保存，评估哪些数据能用于现有示范微调。
- **Opendoor**：保存任务结果和完整执行序列，为后续价值学习留下数据条件；AprilTag 检测事件与人工核验结果应能分别追溯。
- 完整复现 RECAP 需要价值模型、奖励与优势条件训练，不能仅替换 PI0.5 的训练数据列表。

## 精读问题

- [ ] 奖励、终止与时间代价怎样定义？
- [ ] advantage 标签与人工干预标记怎样结合？
- [ ] 不同来源的数据如何采样，哪些用于价值模型，哪些用于策略？
- [ ] 如何把纠正数据收益与 RL 学习收益分开比较？

## 关联

已有 [[concepts/Advantage-conditioned 微调|Advantage-conditioned 微调]]。对照 [[机器人学习的数据策略/论文/HG-DAgger|HG-DAgger]] 与 [[机器人学习的数据策略/论文/Mulligan|Mulligan]] 的数据用法。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
