# 单卡 1.3B 数学推理训练：CPT → SFT → DAPO

> 在一张 GB300 上，把 1.3B 模型的「继续预训练 → SFT → 强化微调」整条链路自己写出来。

**类型** 项目 · **角色** 独立实现 · **时间** 2026 · **状态** 已完成

[← 返回首页](../../README.md)

## 挑战

单卡资源下，既要跑通 10B token 的继续预训练，又要在可接受时间内完成 RL 采样与更新；现成框架（TRL / verl）的单卡效率与可改造成本都不合适。

## 做法

- 基于 PyTorch + Transformers 自研训练与 rollout 循环，不依赖 TRL / verl
- 数学继续预训练：OpenWebMath，10B token 全参训练
- SFT：GSM8K + MATH 共 14,459 条
- DAPO 强化微调：2,652 轮、10,608 次更新
- rollout 优化：回答即时移出解码 batch、microbatch 调优、按历史难度路由选题（75% 历史 + 25% 探索）
- 训练侧使用 Liger 融合 kernel 与 chunked cross-entropy，并对 RMSNorm / SwiGLU / RoPE / CE 做后端对照实验

## 结果

- GSM8K：4.78% → 21.30%（SFT）→ 23.73%（RL 早期检查点）
- MATH：4.16% → 6.14%
- 256 路并发下有效组产出速度约 1.88 倍

## 关键数字

| 指标 | 数值 |
| :-- | :-- |
| GSM8K 准确率 | 4.78% → 23.73% |
| MATH 准确率 | 4.16% → 6.14% |
| 继续预训练规模 | 10B token |
| rollout 有效组产出 | 约 1.88× |
