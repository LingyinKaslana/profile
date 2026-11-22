# 单卡 1.3B 数学推理训练：CPT → SFT → DAPO

> 一张 GB300 跑完三阶段；真正花时间的是两条优化线——训练侧算子融合/编译，和 rollout 采样。

**类型** 项目 · **角色** 独立实现 · **时间** 2026 · **状态** 已完成

[← 返回首页](../../README.md)

## 一、背景

- 1.3B 的 DeepSeek-Coder-1.3B-Base，单卡 GB300（约 251 GiB）：CPT 10B token → SFT → DAPO 2,652 轮。
- GSM8K 4.78% → 8.64% → 21.30% → **23.73%**；MATH 4.16% → 5.64% → 6.14% → 5.34%（RL 早期回落，如实保留）。
- 但这个项目真正想做的是：**单卡上每一段耗时到底被什么占着，哪些优化真的省时间**。

## 二、训练侧：融合 + 编译

- **先有校验，再谈加速**：每个候选后端都要过一遍数值检查——loss 差、全局梯度相对 L2、逐张量 probe 梯度；不通过就不进训练。所以 `runs/cpt_optimization_20261008/` 里是几十个候选，而不是一个「调好的版本」。
- **融合 + torch.compile**：RMSNorm / SwiGLU / RoPE / CE 融进计算图，Adam 也走 compiled AdamW，和 native 实现逐项对照。
- **Liger 内核**：试了 `rms_swiglu_rope`，梯度相对 L2 约 3%、loss 差 1.8e-3，没通过校验，放弃；`liger_lce`（chunked CE，chunk=16）这版通过，保留。
- **chunked CE**：chunk 4 / 16 / 64 扫了一轮，在压显存的同时保住数值。

同模型、同 batch、同精度下，用 Nsight Perf 采计数器对比：

| 指标 | baseline | 优化后 |
| :-- | --: | --: |
| 吞吐（token/s） | 59,110 | **97,367（1.65×）** |
| forward + loss | 184.9 ms | 99.0 ms |
| backward | 334.4 ms | 219.5 ms |
| optimizer | 25.9 ms | 9.4 ms |
| Tensor pipe 活跃 | 29.3% | 54.0% |
| L2 命中率 | 38.4% | 54.1% |
| 每 token 的 HBM 流量 | 55.4 MB | 25.8 MB（−53.5%） |

不是「换了个快实现」那么简单：Tensor pipe 利用率翻倍、HBM 流量减半、L2 命中上升，说明是**算子结构**变了，而不是单纯调大 batch。

## 三、rollout 侧：采样怎么变快

- **完成即移出解码 batch**：不等整批最长序列结束，已完成的回答立刻退出 decode（没做完整 continuous batching）。
- **microbatch 调优**：256 条回答切成 4 个 64-response minibatch，microbatch 取 16。
- **profiling 定位**：先用 profiler 看清 decode 的耗时分布再决定改哪，避免把 CPU 观测和队列等待混进 decode 计时。
- **效果**：最近 100 轮约 **265 秒/轮**（13.6 轮/小时）。
- **按历史难度路由选题**：每题历史正确率做 EMA，75% 优先选「历史上有对有错」的题、25% 随机探索；同权重 256 路采样，uniform 108.23 s vs history 57.63 s，有效组产出约 **1.88 倍**（64 路小并发约 3.50 倍）。

## 四、结果

| 检查点 | GSM8K | MATH |
| :-- | --: | --: |
| 原始 1.3B | 4.78% | 4.16% |
| CPT 10B | 8.64% | 5.64% |
| SFT step 904 | 21.30% | 6.14% |
| 早期 RL 第 80 轮 | **23.73%** | 5.34% |

长训练另用一套口径（256 题 held-out、zero-shot、2,048 token）：最佳第 800 轮 **41.41%**，第 2,600 轮 39.06%。

## 五、局限

- 1.65× 是 Nsight Perf 采样窗口内的吞吐比，不是全程恒定加速；1.88× 也只是短期采样对比，后期大量题目变成全对，倍率会回落。
- 单卡、小模型、小数据的流程实验，不能当作方法有效性的论文级证据；GSM8K 训练题同时用于 SFT 和 RL，不做泛化归因。

## 参考

- [DeepSeekMath-Small（训练工作区快照）](https://github.com/LingyinKaslana/DeepSeekMath-Small)
- [DAPO](https://arxiv.org/abs/2503.14476) · [DeepSeekMath](https://arxiv.org/abs/2402.03300) · [OpenWebMath](https://huggingface.co/datasets/open-web-math/openwebmath)
