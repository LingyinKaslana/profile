# 英伟达半导体科技（上海）｜ PerfWorks / PerfSDK 性能监测团队

> 驱动 GPU 硬件计数器采样、并将原始 counter 解码为指标，为 Nsight System 提供底层支持。

**时间** 2026.03 – 2026.11 · **角色** Software Engineer Intern

[← 返回首页](../../README.md)

## 轻量级 GPU 性能监测（Monitoring）

- 实现跨平台命令行、可配置指标采集与 UI 展示的 GPU 指标监测工具，100 fps 采样下 CPU 占用通常低于 2%，随 Nsight Perf SDK 2026.3 发布。

## Profiler 离线解码（Offline Decode, CUDA 13.5-nsight-perf-sdk-for-ai 的核心之一）

- 定位到采样与搬运开销极低、成本几乎全在解码：解码与被测负载争抢同一 CPU，既干扰观测，又拖慢采样。
- 将解码移出采集路径——采集侧只把 counter 落成 Snapshot，事后重建 Decoder 离线解码，采样不再受解码拖累，可全速跑满全部 counter。
- 解决复现问题：重采拿不到相同数据、问题难以复现，离线解码使排查回到同一份原始数据。

## 存量代码的 AI 审查（AI Code Review）

- 存量千万行 C/C++ 的全量 AI 审查不可行——可疑点远超人工确认能力；提出只筛高置信、与业务无关的局部缺陷，经 10 万行抽样验证，检出 6 个高置信 bug、成本不到 1 美元，外推全仓不到 500 美元，极大减轻人工排查压力。

## 相关链接

- 发布说明（Nsight Perf SDK 2026.3）：https://developer.nvidia.com/nsight-perfsdk/getting-started/release-note-v2026.3
- 下载：https://developer.nvidia.com/nsight-perf-sdk
