---
title: "A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning"
source: "https://proceedings.mlr.press/v15/ross11a.html"
authors: ["Stéphane Ross","Geoffrey J. Gordon","J. Andrew Bagnell"]
year: "2011"
status: preliminary
pdf_source: "https://proceedings.mlr.press/v15/ross11a/ross11a.pdf"
pdf_version: "AISTATS 2011 / PMLR 15"
pdf_pages: 9
downloaded: 2026-11-06
category: "人工干预与纠正"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# DAgger

**原文标题：** A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/DAgger.pdf|本地 PDF]] · [原文与版本](https://proceedings.mlr.press/v15/ross11a.html)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：为什么只增加正常示范，可能仍然无法覆盖策略实际运行时遇到的状态？**

## 问题与核心思路

策略的动作会改变后续状态，因此测试输入分布依赖策略本身。DAgger 让策略运行，在访问到的状态上查询专家动作，将新标注并入数据集后重新训练。[论文](https://proceedings.mlr.press/v15/ross11a.html)

## 方法与数据

1. 用当前策略与专家的迭代混合策略收集轨迹。
2. 对访问到的状态取得专家动作标签。
3. 聚合历轮数据，训练下一轮策略。

这里的关键是状态分布和专家标签；不要求先发生失败才请求监督，也不等同于人只在危险时接管。训练标签不是机器人自己执行的错误动作。[原文 Algorithm 3.1](https://proceedings.mlr.press/v15/ross11a/ross11a.pdf)

## 实验与边界

论文给出在线学习归约和相关保证，并在序列预测及模仿学习任务上验证。理论需要对应的专家、损失及学习假设；不能直接转化为某个真实机械臂任务的成功率保证。对人类专家，持续提供未实际执行的动作标签存在交互困难，这也是后续 HG-DAgger 讨论的问题。[原论文](https://proceedings.mlr.press/v15/ross11a.html) · [HG-DAgger](https://arxiv.org/abs/1810.02890v2)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：记录策略偏离示教后出现的状态，再判断是否有相应正确动作监督。
- **Opendoor**：补采应考虑实际策略在接近和接触阶段访问的状态，而不仅是增加相同 RESET 起点的完整示范。

## 精读问题

- [ ] 专家混合比例怎样影响状态分布？
- [ ] 专家查询动作与实机执行动作怎样分别存储？
- [ ] 人工只能间歇接管时，哪些理论或数据假设发生变化？

## 关联

[[机器人学习的数据策略/论文/HG-DAgger|HG-DAgger]]是人类接管协议；[[机器人学习的数据策略/论文/DART|DART]]从扰动示教的方向处理分布偏移。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
