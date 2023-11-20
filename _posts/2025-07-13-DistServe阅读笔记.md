---
title: DistServe阅读笔记
author: Zhang Ge
date: 2025-07-13 15:46:00 +0800
categories: [专业积累, 编程积累]
tags: [LLM]
pin: false
---

论文原文：https://arxiv.org/abs/2401.09670

# Why PD Disag make sense


independent tuing

# (KEY) How to get PD Disag profit

# Limitations/not suggested situations
1. 优化throughut的场景：PD分离目标在于时延。换句话说，如果TTFT和TPOT优化得更好，那么在给定的TTFT和TPOT约束下，就可以达到更高的request rate。如果目标是最大化throughput（比如offline inference），那么不分离的chunked prefill更适合，因为可以把GPU打得更满。
2. GPU资源有限的场景：PD分离后的设计需要精心设计。GPU数量少，PD分离的设计空间（比如PD配比，并行方式）就会受限。（很）有可能分离后性能反而下降。
3. 长context场景：长context对应更大的KV传输开销，有可能反而降低整体性能。

# Experiment result
阿里云给了一些实验数据：
https://help.aliyun.com/zh/pai/user-guide/enable-prefill-decode-disaggregation-deployment-of-llm-services
