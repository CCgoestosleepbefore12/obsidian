---
title: "HG-DAgger: Interactive Imitation Learning with Human Experts"
source: "https://arxiv.org/abs/1810.02890v2"
authors: ["Michael Kelly","Chelsea Sidrane","Katherine Driggs-Campbell","Mykel J. Kochenderfer"]
year: "2019"
status: preliminary
pdf_source: "https://export.arxiv.org/pdf/1810.02890v2"
pdf_version: "arXiv 1810.02890v2"
pdf_pages: 7
downloaded: 2026-11-06
category: "人工干预与纠正"
updated: 2026-11-06
tags: [data, imitation-learning]
---

# HG-DAgger

**原文标题：** HG-DAgger: Interactive Imitation Learning with Human Experts

[[机器人学习的数据策略/首页|专题首页]] · [[机器人学习的数据策略/附件/HG-DAgger.pdf|本地 PDF]] · [原文与版本](https://arxiv.org/abs/1810.02890v2)

> [!note] 初步整理，待精读
> AI 根据论文与官方资料整理。个人阅读进度尚未确认；项目应用属于候选思路，没有登记为个人结论或已完成实验。

**阅读问题：怎样通过明确的人工接管，让收集到的纠正动作更可靠？**

## 问题与核心思路

原始 DAgger 要求专家给出策略访问状态上的动作标签，但人未必能在不实际控制系统时提供高质量标签。HG-DAgger 让人决定何时接管，在掌握控制权时提供示范。[论文](https://arxiv.org/abs/1810.02890v2)

## 方法与数据

- 策略自主执行；专家认为需要干预时接管，再交回控制。
- 用人实际控制期间的状态与动作补充训练数据。
- 方法还利用干预时的模型不确定性信息学习风险阈值。
- 收集人控数据和记录机器人自主轨迹应区分；不能把接管前的错误动作自动当作专家监督。[原文方法](https://arxiv.org/abs/1810.02890v2)

## 实验与边界

论文使用仿真与真实自动驾驶任务，对比 BC 和 DAgger。这里验证的是其具体系统与专家交互条件，迁移到双臂精细操作还需要检验接管延迟和控制连续性。预印本首次发表于2018年，会议版本为 ICRA 2019。[原文](https://arxiv.org/abs/1810.02890v2)

## 对两个项目的候选启发

以下是 AI 提出的待验证迁移思路：

- **Remove tape**：先确认同构臂能否可靠地从自主执行切到人工控制，并记录接管起止及反馈，不能把已有重抓示范直接称为 HG-DAgger 数据。
- **Opendoor**：检查自主服务、遥操和控制器之间的交接；接管段须能与任务阶段、模型版本对齐。

## 精读问题

- [ ] 何时接管与何时交还，原论文怎样定义？
- [ ] 风险阈值的训练和部署用途是什么？
- [ ] 接管瞬间的过渡帧是否适合作为动作监督？
- [ ] 比较方法时怎样计入人的监控时间与接管时间？

## 关联

[[机器人学习的数据策略/论文/DAgger|DAgger]] · [[机器人学习的数据策略/论文/RaC|RaC]] · [[机器人学习的数据策略/论文/RECAP|RECAP]]

## 阅读后的个人结论

待实际阅读后填写：是否采用、理由、证据强度，以及下一项阅读或实验行动。
