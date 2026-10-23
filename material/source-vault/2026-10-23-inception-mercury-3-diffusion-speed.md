---
title: "Inception's Mercury 3 generates 1,068 tokens per second with diffusion, 4-8x faster than rivals"
date: "2026-10-23"
category: "Tools"
source: "Inception Labs / The Rundown AI"
url: "https://www.inceptionlabs.ai/blog/introducing-mercury-3"
status: "New"
tags: ["inception", "mercury", "diffusion-models", "inference-speed", "llm"]
---

## Summary

Inception launched Mercury 3 on October 19, a diffusion language model the company says generates 1,068 output tokens per second on NVIDIA GPUs — four to eight times the speed of OpenAI's GPT-6 Luna and Anthropic's Claude Haiku 5.5. Instead of building text one token at a time like autoregressive models, Mercury drafts many tokens at once and refines them, an approach Inception has been developing since its 2025 Mercury research.

The model is 40x larger than Mercury 1 in total parameters while keeping similar speed. Quality claims are more modest: Inception's own benchmark table shows mixed results, with Luna and Haiku each winning some tests at medium effort. Pricing is aggressive — $0.20 per million input tokens and $0.70 per million output, halved to $0.10/$0.35 during a launch discount with no stated end date.

The real target is interactive apps where lag kills the experience: live tutors, voice support, browsing agents. OpenAI is chasing the same prize from the hardware side — GPT-6 Astra Ultrafast hits 300 tokens per second in Codex, and the Cerebras deal adds 750 MW of fast-response compute through 2028.

## Key Points

- 1,068 tokens/sec on NVIDIA GPUs; 4-8x faster than GPT-6 Luna and Claude Haiku 5.5
- Diffusion architecture drafts whole blocks of tokens at once instead of one-by-one
- 40x more parameters than Mercury 1 at similar speed; benchmarks show mixed quality vs. speed-class rivals
- Pricing: $0.20/$0.70 per million tokens (input/output), 50% launch discount
- Best fit: latency-sensitive apps — voice agents, live tutoring, browser agents

## Blog Angle

Speed is becoming a product feature, not a spec-sheet number. Write about the latency war: diffusion models (Mercury) vs. custom hardware (OpenAI+Cerebras) as two different bets on making AI feel instant. Practical angle for builders — when 1,000 tokens/sec actually matters (voice, agents) vs. when it doesn't (deep reasoning), and how to split traffic between a fast model and a frontier model.
