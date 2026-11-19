# 潘小康

> 研究 × 系统：让扩散生成模型更快落地，让 GPU 的每一毫秒都可见。

华东师范大学 · 计算机科学与技术 · 硕士在读（DMCV 实验室）

`51285901019@ecnu.stu.edu.cn` · `137 9594 3107` · [GitHub](https://github.com/LingyinKaslana) · [简历 PDF](assets/resume.pdf)

---

## 一眼看

| 3 篇一作 | 50× 加速 | GSM8K 4.78% → 23.73% | 100 fps / CPU < 2% | 竞赛 2 金 1 银 |
| :-- | :-- | :-- | :-- | :-- |
| ICLR 27 在投 · AAAI 27 在投 · CVPR 26 Findings | Fast-HOI，20 步 vs 上千步 | 单卡 1.3B，CPT→SFT→DAPO | NVIDIA GPU 指标监控 | CCPC ×2 · ICPC EC Final · 百度之星 |

## 目录

- [能力](#能力)
- [经历](#经历)
- [代表项目](#代表项目)
- [论文](#论文)
- [竞赛](#竞赛)
- [联系](#联系)

---

## 能力

**LLM 训练与系统** — 全流程自研，不依赖 TRL / verl
继续预训练 → SFT → DAPO 强化微调全链路；RL 采样策略设计、长度与梯度异常诊断；单卡训练与 rollout 加速（Liger、chunked CE）。

**GPU 与性能工程** — 会写 kernel，也看得懂 counter
CUDA 并行：Blelloch Scan、stream compaction、bank conflict；Nsight Systems 做 kernel 级 profiling；NVIDIA PerfSDK 计数器采样与离线解码。

**生成式视觉研究** — 可控、物理一致、少步生成
可控 HOI：VLM 奖励 + 残差策略后处理；显式物理建模：力 / 力矩与 affordance map；少步蒸馏：20 步逼近上千步质量。

## 经历

| 时间 | 单位 | 说明 |
| :-- | :-- | :-- |
| 2021.09 – 2025.06 | 南京信息工程大学 | 计算机科学与技术，工学学士。校程序设计竞赛队总负责人，负责校内 OnlineJudge 维护、主持校级竞赛命题与举办 |
| 2025.09 – 2028.06 | 华东师范大学 | 计算机科学与技术，硕士在读。DMCV 实验室，预计 2028 年 6 月毕业 |
| 2026.03 – 2026.11 | NVIDIA 英伟达半导体科技（上海） | PerfWorks / PerfSDK 性能监测团队，Software Engineer Intern。驱动 GPU 硬件计数器采样，并把原始 counter 解码为指标，为 Nsight System 提供底层支持 |

## 代表项目

- [单卡 1.3B 数学推理训练：CPT → SFT → DAPO](docs/projects/math-rl.md)
  一张 GB300 上自己写训练与 rollout 循环。GSM8K 4.78% → 23.73%，rollout 有效组产出 1.88×。
- [RL 训练效率与稳定性：采样策略与异常梯度](docs/projects/rl-efficiency.md)
  MATH500 mean@4 85.1% → 89.75%；把裁剪前梯度从 142.31 压到 1 量级。
- [NVIDIA PerfSDK：GPU 计数器离线解码与轻量监控](docs/projects/nvidia-perf.md)
  100 fps 采样下 CPU < 2%；10 万行代码抽样检出 6 个高置信 bug，成本不到 1 美元。
- [CUDA 并行计算：Blelloch Scan 与鸟群仿真](docs/projects/cuda-scan.md)
  前缀和 O(n log n) → O(n)；16,384 只鸟 0.5 fps → 45+ fps。

## 论文

- [TriFactorHOI：基于结构化扩散的可控人-物交互生成](docs/papers/trifactor-hoi.md)
  第一作者 · ICLR 2027 在投。手部穿透 −8.4%，脚部滑动 −21.1%。
- [从隐式到显式：面向手物交互生成的动力学建模方法](docs/papers/explicit-hoi.md)
  第一作者 · AAAI 2027 在投。把力 / 力矩与几何接触线索显式注入扩散框架。
- [Fast-HOI：快速人-物交互生成](docs/papers/fast-hoi.md)
  第一作者 · CVPR 2026 Findings。20 步去噪，速度提升 50× 以上，代码开源。

## 竞赛

- 2022 中国大学生程序设计竞赛（CCPC）· 金奖
- 2023 中国大学生程序设计竞赛（CCPC）· 金奖
- 2022 国际大学生程序设计竞赛亚洲总决赛（ICPC EC Final）· 银奖
- 2023 江苏省大学生程序设计竞赛（JSCPC）· 季军
- 2025 第二十一届百度之星程序设计大赛决赛 · 金奖

## 联系

- 邮箱：`51285901019@ecnu.stu.edu.cn`
- GitHub：[@LingyinKaslana](https://github.com/LingyinKaslana)
- 简历：[简历-潘小康.pdf](assets/resume.pdf)
- 开放方向：LLM 训练系统 · 生成模型推理加速 · GPU 性能工程
