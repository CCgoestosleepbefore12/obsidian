---
title: "Adversarial Data Collection: Human-Collaborative Perturbations for Efficient and Robust Robotic Imitation Learning"
source: "https://arxiv.org/abs/2503.11646v1"
authors: ["Siyuan Huang","Yue Liao","Siyuan Feng","Shu Jiang","Si Liu","Hongsheng Li","Maoqing Yao","Guanghui Ren"]
year: "2025"
status: preliminary
pdf_source: "https://arxiv.org/pdf/2503.11646v1"
pdf_version: "arXiv 2503.11646v1"
pdf_pages: 8
downloaded: 2026-11-06
category: "主动制造困难"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# ADC

**原文标题：** Adversarial Data Collection: Human-Collaborative Perturbations for Efficient and Robust Robotic Imitation Learning

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/ADC.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2503.11646v1)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：怎样在一次示范中引入有意义的变化，提高数据的信息密度？**

## 问题与核心思路

ADC 采用两人协作：一人遥操，另一人改变物体、环境或语言指令，遥操者实时应对。目标是在较少示范中获得更多变化与恢复行为。[论文](https://arxiv.org/abs/2503.11646v1)

## 方法与数据

- 扰动对象包括物体位置／姿态、视觉背景等条件以及任务指令。
- 遥操者面对变化继续完成任务，记录新的观测、指令和动作。
- 动态指令需要与发生时刻一致，不能给整个 episode 贴上后来才出现的目标。
- 这里的对抗采集由人组织，不等于训练一个对抗 RL 策略。[原文 §III](https://arxiv.org/html/2503.11646v1)

## 实验与边界

论文比较静态、动态及传感器故障场景，报告少量 ADC 示范下的鲁棒性收益。摘要中的20%示范量是其实验口径；不能换算成只需要20%人工工时，尤其需要考虑第二位操作者。[论文](https://arxiv.org/abs/2503.11646v1)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：把目标偏位、胶条状态变化等真实失败因素列成候选，先辨别哪些是可恢复变化。
- **Opendoor**：区分物体与环境变化、任务定义变化和系统故障。技术断流不能仅因任务最终成功就视为有效扰动数据。
- 候选管线字段包括扰动类型、时间、参与者、指令版本以及恢复结果。

## 精读问题

- [ ] 扰动怎样分配到不同阶段，是否有统一采集规则？
- [ ] 数据效率按条数、时长、帧数还是人工工时比较？
- [ ] 哪些实验支持恢复能力，哪些主要反映视觉或语言泛化？
- [ ] 相同扰动是否必须出现在评测中才有收益？

## 关联

[[机器人学习的数据策略/论文/DART|DART]]改变示教动作；[[机器人学习的数据策略/论文/Robotic Data Inoculation|Data Inoculation]]学习动作残差。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
