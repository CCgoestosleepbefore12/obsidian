---
title: "Universal Manipulation Interface: In-The-Wild Robot Teaching Without In-The-Wild Robots"
source: "https://arxiv.org/abs/2402.10329v3"
authors: ["Cheng Chi","Zhenjia Xu","Chuer Pan","Eric Cousineau","Benjamin Burchfiel","Siyuan Feng","Russ Tedrake","Shuran Song"]
year: "2024"
status: preliminary
pdf_source: "https://roboticsproceedings.org/rss20/p045.pdf"
pdf_version: "RSS 2024 会议版"
pdf_pages: 14
downloaded: 2026-11-06
category: "基础示教"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# UMI

**原文标题：** Universal Manipulation Interface: In-The-Wild Robot Teaching Without In-The-Wild Robots

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/UMI.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/2402.10329v3)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：采集接口和部署接口怎样共同决定示范能否被机器人有效使用？**

## 问题与核心思路

UMI 用手持夹爪采集人类操作示范，通过观测、动作表示与时延匹配，使这些数据能用于训练可部署的机器人策略。它关注采集接口和迁移，不是一个纠错采集算法。[原文](https://arxiv.org/abs/2402.10329v3)

## 方法与数据

- 采集端采用带相机的手持夹爪，获得视觉、运动轨迹和夹爪开度。
- 相对轨迹表示与部署时延匹配是方法组成部分，不能只复制硬件外形。
- 人类示范被用于模仿学习；UMI 与 BC 属于不同层次，并非替代关系。
- 阅读重点是数据如何成为策略可用的观测和动作，以及怎样检查目标机器人能否执行。[项目与消融](https://real.stanford.edu/umi/)

## 实验与边界

项目展示了动态、双臂、精细和长时序任务，并比较了时延匹配、双夹爪相对信息等设计。它说明接口设计会影响策略表现；不能据此假定任意手持轨迹都能迁移到任意机器人。[项目](https://real.stanford.edu/umi/)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：先比较现有同构臂示教与部署的时间语义、动作表示和可执行节奏，不预设需要换采集硬件。
- **Opendoor**：把视觉、状态、动作的时间关系与部署时延分开记录，检查训练合同是否和执行合同匹配。

## 精读问题

- [ ] 哪些时延在采集端校准，哪些在部署端补偿？
- [ ] 轨迹筛查依赖哪些机器人运动学条件？
- [ ] 相对轨迹的参考系、时间原点和双臂关系怎样定义？

## 关联

[[机器人学习的数据策略/论文/DAgger|DAgger]]讨论策略访问状态上的监督；[[机器人学习的数据策略/论文/Residual Copilot|Residual Copilot]]讨论采集时的辅助控制。

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
