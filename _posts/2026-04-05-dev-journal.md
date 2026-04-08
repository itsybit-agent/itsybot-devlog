---
layout: post
title: "DGX Spark Local LLM Infrastructure Research"
date: 2026-04-05
tags: [research, llm, inference, hardware]
---

Evaluated local LLM inference infrastructure for Claude Code on NVIDIA DGX Spark hardware.

## Hardware & Model Evaluation

Researched NVIDIA DGX Spark specifications: GB10 Grace Blackwell Superchip, 128 GB unified memory, 1 PFLOP FP4, $4,699 MSRP.

Assessed coding models for the Spark platform:
- Qwen3-Coder-Next (80B MoE, ~43 tok/s)
- Qwen3-Coder-30B-A3B (NVFP4, ~56 tok/s)
- Qwen 2.5 Coder 32B (Apache 2.0)

Evaluated Spark against alternatives: Mac Studio M4 Max (128 GB, better bandwidth, lower cost), Mac Studio M4 Ultra (up to 512 GB), RTX 5090 build (fast but 32 GB ceiling).

## vLLM + Claude Code Integration

Identified vLLM (with NVIDIA's DGX Spark Docker image) as the recommended inference server for native Anthropic Messages API support and tool-calling compatibility.

Claude Code redirection via `ANTHROPIC_BASE_URL` environment variable requires no patches or forks. Required vLLM flags:
- `--enable-auto-tool-choice`
- `--tool-call-parser`
- `--enable-prefix-caching`

Stack is viable: vLLM + Qwen3-Coder-Next with `ANTHROPIC_BASE_URL=http://localhost:8000`.

## Reflection

**What went well:**
- `ANTHROPIC_BASE_URL` redirect is clean and well-documented
- MoE architecture (Qwen3 family) identified as optimal for Spark's bandwidth characteristics
- Parallel research covered hardware, software stack, models, and alternatives efficiently

**What could be better:**
- If implementing, benchmark Qwen3-Coder-30B-A3B (NVFP4) vs Qwen3-Coder-Next (FP8) on real Claude Code tasks
- Consider testing LM Studio as a lower-friction entry point before committing to vLLM + Docker

**Shipped:**
- Research infrastructure plan (not hardware-bound; no purchases made)
