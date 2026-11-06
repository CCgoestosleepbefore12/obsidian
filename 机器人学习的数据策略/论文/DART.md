---
title: "DART: Noise Injection for Robust Imitation Learning"
source: "https://proceedings.mlr.press/v78/laskey17a.html"
authors: ["Michael Laskey","Jonathan Lee","Roy Fox","Anca Dragan","Ken Goldberg"]
year: "2017"
status: preliminary
pdf_source: "https://proceedings.mlr.press/v78/laskey17a/laskey17a.pdf"
pdf_version: "CoRL 2017 / PMLR 78"
pdf_pages: 14
downloaded: 2026-11-06
category: "主动制造困难"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# DART

**原文标题：** DART: Noise Injection for Robust Imitation Learning

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/DART.pdf|本地 PDF]] · [原文与版本](https://proceedings.mlr.press/v78/laskey17a.html)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：能否在正常示教过程中主动引出偏差，让人示范如何恢复？**

## 问题与核心思路

普通示范容易只覆盖专家顺利操作的状态。DART 在专家示教动作中注入噪声，使采集过程包含偏离与恢复，并调整噪声以近似学习策略的误差。[论文](https://proceedings.mlr.press/v78/laskey17a.html)

## 方法与数据

- 使用带噪声的专家控制访问更广的状态，专家持续闭环应对。
- 迭代采集、训练策略，再根据策略与专家的差异调整噪声参数。
- 数据包含扰动下状态上的专家动作；扰动动作与专家标签的具体实现要分别核对。
- 不是对已录好的关节轨迹随意加噪声：真实扰动会改变后续观测和专家反应。[原文算法](https://proceedings.mlr.press/v78/laskey17a/laskey17a.pdf)

## 实验与边界

论文包含 MuJoCo 控制任务及 Toyota HSR 杂乱环境抓取实验，并与 BC、DAgger 比较。噪声模型和专家能够恢复的范围影响适用性；接触操作中不能从某个任务照搬幅度。[论文](https://proceedings.mlr.press/v78/laskey17a.html)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：先判断真实误差出现在哪个阶段，再研究对应的偏差和恢复数据，不把全程抖动等同于有效覆盖。
- **Opendoor**：区分动作偏差与导航起点偏差；两者改变状态分布的方式不同。
- 如设计候选采集，需要把扰动前指令、实际执行、观测及后续专家动作关联保存。

## 精读问题

- [ ] 噪声分布怎样估计，是否随状态和任务阶段变化？
- [ ] 专家监督与带噪执行动作如何区分？
- [ ] 哪些噪声只增加难度，却不增加部署相关覆盖？

## 关联

[[机器人学习的数据策略/论文/ADC|ADC]]由人改变环境和任务条件；[[机器人学习的数据策略/论文/Robotic Data Inoculation|Data Inoculation]]使用在线学习的残差。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
